private void csvSalesOutDataToXLSX() {

&nbsp;   try (XSSFWorkbook workBook = new XSSFWorkbook()) {

&nbsp;       File fileSales = new File(ECPBatchConstant.ECP\_SALES\_OUTPUT);

&nbsp;       String csvSaleFileAddress = fileSales.toPath().toString();

&nbsp;       String xlsxFileAddress = "ecp\_batch\_" + LocalDate.now().getMonthValue() + LocalDate.now().getDayOfMonth()

&nbsp;               + LocalDate.now().getYear() + ".xlsx";

&nbsp;       

&nbsp;       XSSFSheet sheet = workBook.createSheet("sheet1");

&nbsp;       String currentLine = null;

&nbsp;       int RowNum = 0;

&nbsp;       try (BufferedReader br = new BufferedReader(new FileReader(csvSaleFileAddress))) {

&nbsp;           while ((currentLine = br.readLine()) != null) {

&nbsp;               String str\[] = currentLine.split("\\\\|");

&nbsp;               RowNum++;

&nbsp;               XSSFRow currentRow = sheet.createRow(RowNum);

&nbsp;               for (int i = 0; i < str.length; i++) {

&nbsp;                   currentRow.createCell(i).setCellValue(str\[i]);

&nbsp;               }

&nbsp;           }

&nbsp;       } catch (FileNotFoundException e) {

&nbsp;           LOG.error("Error Reading the sales out csv file.", e);

&nbsp;       }

&nbsp;       

&nbsp;       FileOutputStream fileOutputStream = new FileOutputStream(xlsxFileAddress);

&nbsp;       workBook.write(fileOutputStream);

&nbsp;       workBook.close();

&nbsp;       fileOutputStream.close();

&nbsp;       

&nbsp;       File excelSaleFile = new File(xlsxFileAddress);

&nbsp;       InputStream targetStreamExcelSale = new FileInputStream(excelSaleFile);

&nbsp;       try {

&nbsp;           s3Service.write(this.outputBucketName, xlsxFileAddress, targetStreamExcelSale);

&nbsp;           // deleting temporary copy of output files.

&nbsp;           Files.deleteIfExists(excelSaleFile.toPath());

&nbsp;           Files.deleteIfExists(Paths.get(ECPBatchConstant.ECP\_FEED\_FILE));

&nbsp;           Files.deleteIfExists(new File(ECPBatchConstant.ECP\_BASE\_OUTPUT).toPath());

&nbsp;           Files.deleteIfExists(fileSales.toPath());

&nbsp;       } catch (AmazonServiceException e) {

&nbsp;           LOG.error("Exception occurred while writing xlsx to S3", e);

&nbsp;       }

&nbsp;   } catch (Exception e) {

&nbsp;       LOG.error("Exception occurred while converting file to xlsx.", e);

&nbsp;   }

}

