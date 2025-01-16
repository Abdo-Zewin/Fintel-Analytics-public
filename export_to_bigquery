function exportToBigQuery() {
  // Replace with your Google Sheet and BigQuery details
  var sheetName = "tab_name"; // Name of the tab in your Google Sheet
  var projectId = "BigQuery PROJECT ID"; // GCP Project ID
  var datasetId = "DESTINATION DATASET ID"; // BigQuery Dataset ID
  var tableId = "fx_currencies"; // BigQuery Table Name

  // Open the sheet and fetch data
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName(sheetName);
  var data = sheet.getDataRange().getValues();

  data.shift(); // Remove the first row (headers)

  // Add ingestion timestamp to each row
  var currentTimestamp = new Date().toISOString(); // Get the current timestamp in ISO format
  data = data.map(function (row) {
    return row.concat(currentTimestamp); // Append the timestamp to each row
  });

  // Generate the CSV string
  var rowsCSV = data
    .map(function (row) {
      return row.join(","); // Join each row (array) into a string, separating cells with commas
    })
    .join("\n"); // Join all rows into a single string, separating rows with newlines

  var blob = Utilities.newBlob(rowsCSV, "text/csv");
  var dataBlob = blob.setContentType("application/csv");

  // Create the data upload job
  var job = {
    configuration: {
      load: {
        destinationTable: {
          projectId: projectId,
          datasetId: datasetId,
          tableId: tableId,
        },
        skipLeadingRows: 0, // Skip no rows as we include the timestamp directly
        writeDisposition: "WRITE_APPEND",
      },
    },
  };

  // Send the job to BigQuery so it will run your query
  var runJob = BigQuery.Jobs.insert(job, projectId, dataBlob);
  Logger.log(runJob.status);
  var jobId = runJob.jobReference.jobId;
  Logger.log("jobId: " + jobId);
  var status = BigQuery.Jobs.get(projectId, jobId);

  // Wait for the query to finish running before you move on
  while (status.status.state === "RUNNING") {
    Utilities.sleep(500);
    status = BigQuery.Jobs.get(projectId, jobId);
    Logger.log("Status: " + status);
  }
  Logger.log("FINISHED!");
}
