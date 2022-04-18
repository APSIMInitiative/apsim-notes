# Performance Tests

The apsim [performance tests website](https://apsimdev.apsim.info/APSIM.POStats) tracks the performance (in terms of predicted vs observed accuracy) over time. When a pull request is tested, stats such as RMSE will be calculated for each predicted vs observed datapoint, and the data is submitted to the POStats API. The website itself allows users to explore the stats with graphs/tables and drill down to see which simulations' results have been affected by the changes introduced by a pull request. All of the code for the POStats lives in the [POStats](https://github.com/APSIMInitiative/APSIM.PerformanceTests) repository.

Note that there was a change in nomenclature (around 2020) from PerformanceTests to POStats (or APSIM.PerformanceTests to APSIM.POStats).

## Collector

The POStats collector reads the outputs from each of the apsim validation files and submits the outputs/stats to the API. The collector is a .net console application and is built and run automatically as part of the [apsim test suite](JENKINS.md).

## API

The POStats REST API allows the collector to upload its results. The API will compute some statistics from these results and store them in the database. At any point in time there is an "accepted" value for each of these statistics. If any of the statistics have regressed (e.g. if RMSE increases for a variable), then the overall build is failed. The POStats API wil set a custom status check on the github pull request; this will be set to pass if the status are the same or better than the accepted values, or it will be set to worse if any the stats are worse than the accepted values.

The data is currently all stored in an SQL server database on the apsimdev server, which also currently hosts the API and website.

## Website

The website is linked to by the github status check created by the API. It allows users to explore the changes to the stats introduced by their pull request.
