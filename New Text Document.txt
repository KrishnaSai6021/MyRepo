void processMessage(String message, Acknowledgement ack) {
    //todo
    try {
        log.info("processing message:{}", message);
        JsonNode root = objectMapper.readTree(message);
        JsonNode records = root.path("Records");
        
        if (!records.isArray() || records.size() == 0) { //records.isEmpty()
            log.warn("No records[] found in s3 event; skipping message{}", message);
            return;
        }
        
        JsonNode rec = records.get(0);
        String bucket = rec.path("s3").path("bucket").path("name").asText(null);
        String bucket = rec.path("bucket").path("name").asText();
        String keyEnc = rec.path("s3").path("object").path("key").asText();
        if (bucket == null || keyEnc == null) {
            log.warn("missing bucket/key in s3event;skipping message{}", message);
            return;
        }
        
        String key = URLDecoder.decode(keyEnc, StandardCharsets.UTF_8);
        log.info("Event object:s3://{}/{}", bucket, keyEnc);
        
        System.setProperty("ECP_INPUTKEY", keyEnc);
        System.setProperty("INPUT_BUCKET", bucket);
        
        JobParameters params = new JobParametersBuilder()
                .addString("jobId", UUID.randomUUID().toString())
                .addLong("timestamp", System.currentTimeMillis()).toJobParameters();
        
        if(jobLauncher != null) {
            
            JobExecution launched = jobLauncher.run(ecpBatch, params);
            
            JobExecution finalState = waitForCompletion(launched.getId());
            
            if (finalState.getStatus() == BatchStatus.COMPLETED) {
                ack.acknowledge();
                log.info("Job Completed.deleted SQS message {}", message);
            } else {
                log.warn("Job ended with status {}.keeping message {} for retry.", finalState.getStatus(), message);
            }
        }
        
    } catch (Exception ex) {
        log.error("Failed handling message{}.leaving it for retry.", message, ex);
    }
    try {
        Thread.sleep(1000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}