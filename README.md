```bash
npm i --save request request-promise cheerio
```

```jsx
const request = require("request-promise");
const cheerio = require("cheerio");

const url = "https://sfbay.craigslist.org/d/software-qa-dba-etc/search/sof";

const scrapeSample = {
  title: "Technical Autonomous Vehicle Trainer",
  description:
    "We're the driverless car company. We're building the world's best autonomous vehicles to safely connect people to the places, things, and experiences they care about.",
  datePosted: new Date("2018-07-13"),
  url:
    "https://sfbay.craigslist.org/sfc/sof/d/technical-autonomous-vehicle/6642626746.html",
  hood: "(SOMA / south beach)",
  address: "1201 Bryant St.",
  compensation: "23/hr"
};

const scrapeResults = [];

async function scrapeJobHeader() {
  try {
    const htmlResult = await request.get(url);
    const $ = await cheerio.load(htmlResult);

    /* children vs find */
    //children() only travels a single level down the DOM tree while . 
    //find() can traverse down multiple levels to select descendant elements (grandchildren, etc.) as well.

    $(".result-info").each((index, element) => {
      const resultTitle = $(element).children(".result-title");
      const title = resultTitle.text();
      const url = resultTitle.attr("href");
      const datePosted = new Date(
        $(element)
          .children("time")
          .attr("datetime")
      );
      const hood = $(element)
        .find(".result-hood")
        .text();
      const scrapeResult = { title, url, datePosted, hood };
      scrapeResults.push(scrapeResult);
    });
    return scrapeResults;
  } catch (err) {
    console.error(err);
  }
}

async function scrapeDescription(jobsWithHeaders) {
  return await Promise.all(
    jobsWithHeaders.map(async job => {
      try {
        const htmlResult = await request.get(job.url);
        const $ = await cheerio.load(htmlResult);
        $(".print-qrcode-container").remove(); // you can remove some unwanted element.
        job.description = $("#postingbody").text();
        job.address = $("div.mapaddress").text();
        const compensationText = $(".attrgroup")
          .children()
          .first()
          .text();
        job.compensation = compensationText.replace("compensation: ", "");
        return job;
      } catch (error) {
        console.error(error);
      }
    })
  );
}

async function scrapeCraigslist() {
  const jobsWithHeaders = await scrapeJobHeader();
  const jobsFullData = await scrapeDescription(jobsWithHeaders);
}

scrapeCraigslist();
```

# Parsing into CSV

### Dependency

```bash
npm i objects-to-csv --save
```

### Object to CSV

```jsx
const ObjectsToCsv = require('object-to-csv')

async function createCSVFile(data) {
  let csv = new ObjectsToCsv(data)
  await csv.toDisk('./test.csv')
  console.log(await csv.toString())
}
```

### Entire code

```jsx
const request = require("request-promise");
const cheerio = require("cheerio");
const ObjectsToCsv = require('object-to-csv')

const url = "https://sfbay.craigslist.org/d/software-qa-dba-etc/search/sof";

const scrapeSample = {
  title: "Technical Autonomous Vehicle Trainer",
  description:
    "We're the driverless car company. We're building the world's best autonomous vehicles to safely connect people to the places, things, and experiences they care about.",
  datePosted: new Date("2018-07-13"),
  url:
    "https://sfbay.craigslist.org/sfc/sof/d/technical-autonomous-vehicle/6642626746.html",
  hood: "(SOMA / south beach)",
  address: "1201 Bryant St.",
  compensation: "23/hr"
};

const scrapeResults = [];

async function scrapeJobHeader() {
  try {
    const htmlResult = await request.get(url);
    const $ = await cheerio.load(htmlResult);

    $(".result-info").each((index, element) => {
      const resultTitle = $(element).children(".result-title");
      const title = resultTitle.text();
      const url = resultTitle.attr("href");
      const datePosted = new Date(
        $(element)
          .children("time")
          .attr("datetime")
      );
      const hood = $(element)
        .find(".result-hood")
        .text();
      const scrapeResult = { title, url, datePosted, hood };
      scrapeResults.push(scrapeResult);
    });
    return scrapeResults;
  } catch (err) {
    console.error(err);
  }
}

async function scrapeDescription(jobsWithHeaders) {
  return await Promise.all(
    jobsWithHeaders.map(async job => {
      try {
        const htmlResult = await request.get(job.url);
        const $ = await cheerio.load(htmlResult);
        $(".print-qrcode-container").remove();
        job.description = $("#postingbody").text();
        job.address = $("div.mapaddress").text();
        const compensationText = $(".attrgroup")
          .children()
          .first()
          .text();
        job.compensation = compensationText.replace("compensation: ", "");
        return job;
      } catch (error) {
        console.error(error);
      }
    })
  );
}

//parsing DATA into CSV
async function createCSVFile(data) {
  let csv = new ObjectsToCsv(data)
  await csv.toDisk('./test.csv')
  console.log(await csv.toString())
}

async function scrapeCraigslist() {
  const jobsWithHeaders = await scrapeJobHeader();
  const jobsFullData = await scrapeDescription(jobsWithHeaders);
  //parsing DATA into CSV
  await createCSVFile(jobsFullData)
}

scrapeCraigslist();
```

# Bad Network

```bash
npm i --save requestretry
```

```jsx
const request = require("request-retry").defaults({fullResponse: false});
```

```kotlin
// const request = require("request-promise");

const request = require('requestretry').defaults({fullResponse: false}) //full response => false, it only returns HTML String.

const cheerio = require("cheerio");
const ObjectsToCsv = require('object-to-csv')

const url = "https://sfbay.craigslist.org/d/software-qa-dba-etc/search/sof";

const scrapeSample = {
  title: "Technical Autonomous Vehicle Trainer",
  description:
    "We're the driverless car company. We're building the world's best autonomous vehicles to safely connect people to the places, things, and experiences they care about.",
  datePosted: new Date("2018-07-13"),
  url:
    "https://sfbay.craigslist.org/sfc/sof/d/technical-autonomous-vehicle/6642626746.html",
  hood: "(SOMA / south beach)",
  address: "1201 Bryant St.",
  compensation: "23/hr"
};

const scrapeResults = [];

async function scrapeJobHeader() {
  try {
    const htmlResult = await request.get(url);
    const $ = await cheerio.load(htmlResult);

    /* children vs find */
    //children() only travels a single level down the DOM tree while . 
    //find() can traverse down multiple levels to select descendant elements (grandchildren, etc.) as well.

    $(".result-info").each((index, element) => {
      const resultTitle = $(element).children(".result-title");
      const title = resultTitle.text();
      const url = resultTitle.attr("href");
      const datePosted = new Date(
        $(element)
          .children("time")
          .attr("datetime")
      );
      const hood = $(element)
        .find(".result-hood")
        .text();
      const scrapeResult = { title, url, datePosted, hood };
      scrapeResults.push(scrapeResult);
    });
    return scrapeResults;
  } catch (err) {
    console.error(err);
  }
}

async function scrapeDescription(jobsWithHeaders) {
  return await Promise.all(
    jobsWithHeaders.map(async job => {
      try {
        const htmlResult = await request.get(job.url);
        const $ = await cheerio.load(htmlResult);
        $(".print-qrcode-container").remove(); // you can remove some unwanted element.
        job.description = $("#postingbody").text();
        job.address = $("div.mapaddress").text();
        const compensationText = $(".attrgroup")
          .children()
          .first()
          .text();
        job.compensation = compensationText.replace("compensation: ", "");
        return job;
      } catch (error) {
        console.error(error);
      }
    })
  );
}

async function scrapeCraigslist() {
  const jobsWithHeaders = await scrapeJobHeader();
  const jobsFullData = await scrapeDescription(jobsWithHeaders);
  await createCSVFile(jobsFullData)
}

//parsing DATA into CSV
async function createCSVFile(data) {
  let csv = new ObjectsToCsv(data)
  await csv.toDisk('./test.csv')
  console.log(await csv.toString())
}

scrapeCraigslist();
```