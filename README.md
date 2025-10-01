void processMessage(String message, Acknowledgement ack) {

&nbsp;   //todo

&nbsp;   try {

&nbsp;       log.info("processing message:{}", message);

&nbsp;       JsonNode root = objectMapper.readTree(message);

&nbsp;       JsonNode records = root.path("Records");

&nbsp;       

&nbsp;       if (!records.isArray() || records.size() == 0) { //records.isEmpty()

&nbsp;           log.warn("No records\[] found in s3 event; skipping message{}", message);

&nbsp;           return;

&nbsp;       }

&nbsp;       

&nbsp;       JsonNode rec = records.get(0);

&nbsp;       String bucket = rec.path("s3").path("bucket").path("name").asText(null);

&nbsp;       String bucket = rec.path("bucket").path("name").asText();

&nbsp;       String keyEnc = rec.path("s3").path("object").path("key").asText();

&nbsp;       if (bucket == null || keyEnc == null) {

&nbsp;           log.warn("missing bucket/key in s3event;skipping message{}", message);

&nbsp;           return;

&nbsp;       }

&nbsp;       

&nbsp;       String key = URLDecoder.decode(keyEnc, StandardCharsets.UTF\_8);

&nbsp;       log.info("Event object:s3://{}/{}", bucket, keyEnc);

&nbsp;       

&nbsp;       System.setProperty("ECP\_INPUTKEY", keyEnc);

&nbsp;       System.setProperty("INPUT\_BUCKET", bucket);

&nbsp;       

&nbsp;       JobParameters params = new JobParametersBuilder()

&nbsp;               .addString("jobId", UUID.randomUUID().toString())

&nbsp;               .addLong("timestamp", System.currentTimeMillis()).toJobParameters();

&nbsp;       

&nbsp;       if(jobLauncher != null) {

&nbsp;           

&nbsp;           JobExecution launched = jobLauncher.run(ecpBatch, params);

&nbsp;           

&nbsp;           JobExecution finalState = waitForCompletion(launched.getId());

&nbsp;           

&nbsp;           if (finalState.getStatus() == BatchStatus.COMPLETED) {

&nbsp;               ack.acknowledge();

&nbsp;               log.info("Job Completed.deleted SQS message {}", message);

&nbsp;           } else {

&nbsp;               log.warn("Job ended with status {}.keeping message {} for retry.", finalState.getStatus(), message);

&nbsp;           }

&nbsp;       }

&nbsp;       

&nbsp;   } catch (Exception ex) {

&nbsp;       log.error("Failed handling message{}.leaving it for retry.", message, ex);

&nbsp;   }

&nbsp;   try {

&nbsp;       Thread.sleep(1000);

&nbsp;   } catch (InterruptedException e) {

&nbsp;       Thread.currentThread().interrupt();

&nbsp;   }

}

