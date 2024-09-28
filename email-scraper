const fs = require('fs');
const request = require('request');
const cheerio = require('cheerio');

const MAX_CONCURRENT_REQUESTS = 1000; // Maximum number of concurrent requests
const OUTPUT_FILE = 'output-emails.csv'; // File to write the scraped email addresses to
const TIMEOUT = 15000; // Timeout for each request, in milliseconds

// Queue to control the number of concurrent requests
const requestQueue = new Set();

// Scrape the email address from the given URL
function getEmail() {
var url = "https://learn.coachmariawendt.com/24-for-24/"
  // Check if the URL is valid
  if (!url || !url.startsWith('http')) {
    return null;
  }

  // Add the URL to the queue
  requestQueue.add(url);

  // Check if we can send another request
  if (requestQueue.size < MAX_CONCURRENT_REQUESTS) {
    // Set a timeout to cancel the request after the specified time
    const requestTimeout = setTimeout(() => {
      // Remove the URL from the queue
      requestQueue.delete(url);
    }, TIMEOUT);

    // Send the request
    request(url, (error, response, html) => {
      // Clear the timeout
      clearTimeout(requestTimeout);

      // Remove the URL from the queue
      requestQueue.delete(url);

      if (!error && response.statusCode === 200) {
        const $ = cheerio.load(html);

        // Find the element with the "mailto:" link
        const emailLink = $('a[href^="mailto:"]');

        if (emailLink.length > 0) {
          // Get the email address by removing the "mailto:" part
          const email = emailLink.attr('href').replace('mailto:', '');

          // Write the email address to the file
          fs.appendFile(OUTPUT_FILE, email + '\n', (error) => {
            if (error) {
              // Handle errors
              console.error(error);
            } else {
              // Confirm that the email address was added to the file
              console.log(`Email address added: ${email}`);
            }
          });
        }
      } else {
        // Handle errors
        console.error(error);
      }
    });
  } else {
    // If we've reached the maximum number of concurrent requests, wait a little before checking again
    setTimeout(() => getEmail(url), 500);
  }
}

// Read the file with the list of URLs
fs.readFile('websites.csv', 'utf8', (error, data) => {
  if (error) {
    // Handle errors
    console.error(error);
  } else {
    // Split the file into lines
    const urls = data.split('\n');

    // Scrape the email addresses from each URL
    urls.forEach(getEmail);
  }
});
