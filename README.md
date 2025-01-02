# Chrome-Extension-Scraping-FB-Ads-Library
build a robust scraping tool for the Facebook Ads Library. This tool should:

Keyword-based Search: Efficiently search for ads using customizable keywords.
Timeframe Filtering: Allow filtering ads based on their active time period.
Search by country.
Reach Criteria: Filter ads by their reach (Europe only)
Data Extraction: Extract key information from identified ads, including:
Ad creatives (images, videos, text)
Target audience demographics
-----------------
Building a robust web scraping tool for the Facebook Ads Library requires several steps. While Facebook Ads Library API can provide some information, web scraping would typically be against Facebook's terms of service. However, if you plan to use the API responsibly or only scrape publicly accessible information, I can guide you on how to build a Chrome Extension to search ads, filter based on keywords, timeframe, country, and reach, and then extract relevant information.

Here is a conceptual approach to the Chrome Extension that can interact with the Facebook Ads Library webpage and extract key data using custom filters.
Key Features:

    Keyword-based Search: Customizable keywords to search ads.
    Timeframe Filtering: Filter ads by active timeframe.
    Search by Country: Filter ads by the country of operation.
    Reach Criteria: Filter ads based on reach within specific regions.
    Data Extraction: Extract creatives (images, videos, text) and target audience demographics.

Step-by-Step Guide
1. Create the Project Structure

facebook-ads-library-scraper/
├── src/
│   ├── background.js
│   ├── content.js
│   ├── popup.js
│   ├── popup.html
│   ├── style.css
├── manifest.json
└── icons/
    └── icon.png

2. manifest.json

This file defines the metadata, permissions, and configurations for the extension.

{
  "manifest_version": 3,
  "name": "Facebook Ads Library Scraper",
  "version": "1.0",
  "description": "Scrape and filter ads from Facebook Ads Library based on various criteria.",
  "permissions": [
    "activeTab",
    "storage",
    "tabs"
  ],
  "background": {
    "service_worker": "src/background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://www.facebook.com/ads/library/*"],
      "js": ["src/content.js"],
      "css": ["src/style.css"]
    }
  ],
  "action": {
    "default_popup": "src/popup.html",
    "default_icon": {
      "16": "icons/icon.png",
      "48": "icons/icon.png",
      "128": "icons/icon.png"
    }
  }
}

3. Background Script (background.js)

This script can handle communication between the popup and content scripts, store user settings, or handle any persistent data.

// src/background.js

chrome.runtime.onInstalled.addListener(() => {
  console.log("Facebook Ads Library Scraper Extension Installed.");
});

// Listen for messages from the content script to process extracted data
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.type === "extractAdData") {
    // Handle logic to process and store ad data
    console.log("Ad data extracted:", message.data);
    sendResponse({ status: "success" });
  }
});

4. Content Script (content.js)

The content script interacts directly with the Facebook Ads Library page. It will extract the relevant ad data, including creatives and demographics, based on user-defined filters.

// src/content.js

// Wait for the page to load completely
window.addEventListener('load', function () {
  // Add a button to trigger scraping
  const scrapeButton = document.createElement("button");
  scrapeButton.innerText = "Scrape Ads";
  scrapeButton.style.position = "fixed";
  scrapeButton.style.top = "20px";
  scrapeButton.style.right = "20px";
  scrapeButton.style.zIndex = "9999";
  scrapeButton.style.padding = "10px";
  scrapeButton.style.backgroundColor = "#00c0ff";
  scrapeButton.style.color = "white";
  scrapeButton.style.border = "none";
  scrapeButton.style.borderRadius = "5px";
  document.body.appendChild(scrapeButton);

  // Event listener to handle scraping action
  scrapeButton.addEventListener('click', function () {
    const adsData = [];

    // Logic for filtering ads based on user input (using query params, metadata, etc.)
    const ads = document.querySelectorAll('.ad-item'); // Example selector, modify to match Facebook Ads Library structure
    ads.forEach(ad => {
      const adDetails = {
        title: ad.querySelector('.ad-title')?.textContent,
        description: ad.querySelector('.ad-description')?.textContent,
        image: ad.querySelector('img')?.src,
        video: ad.querySelector('video')?.src,
        demographics: ad.querySelector('.ad-demographics')?.textContent, // Example, modify to match the actual structure
      };

      // Filter ads based on keywords, timeframe, country, and reach
      if (adDetails.title && adDetails.description) {
        adsData.push(adDetails);
      }
    });

    // Send extracted data to background script for processing
    chrome.runtime.sendMessage({ type: "extractAdData", data: adsData }, (response) => {
      if (response.status === "success") {
        alert("Ads data extracted successfully!");
      }
    });
  });
});

5. Popup HTML (popup.html)

The popup interface allows users to define their filtering criteria, such as keywords, timeframe, and country.

<!-- src/popup.html -->

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Facebook Ads Scraper</title>
    <link rel="stylesheet" href="style.css">
  </head>
  <body>
    <h1>Ad Scraper Filters</h1>
    <div>
      <label for="keyword">Keyword:</label>
      <input type="text" id="keyword" placeholder="Enter a keyword">
    </div>
    <div>
      <label for="country">Country:</label>
      <select id="country">
        <option value="US">United States</option>
        <option value="UK">United Kingdom</option>
        <option value="DE">Germany</option>
        <!-- Add more countries as needed -->
      </select>
    </div>
    <div>
      <label for="timeframe">Timeframe:</label>
      <input type="date" id="timeframe">
    </div>
    <button id="applyFilters">Apply Filters</button>

    <script src="popup.js"></script>
  </body>
</html>

6. Popup JS (popup.js)

The JS logic to handle filter settings and pass them to the content script for ad scraping.

// src/popup.js

document.getElementById("applyFilters").addEventListener("click", function () {
  const keyword = document.getElementById("keyword").value;
  const country = document.getElementById("country").value;
  const timeframe = document.getElementById("timeframe").value;

  // Send filter data to content script to process scraping
  chrome.storage.local.set({ filters: { keyword, country, timeframe } }, () => {
    alert("Filters applied!");
  });
});

7. Styling (style.css)

A simple style for the popup and the page UI.

/* src/style.css */

body {
  font-family: Arial, sans-serif;
  padding: 10px;
}

h1 {
  font-size: 18px;
}

button {
  background-color: #007bff;
  color: white;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

input, select {
  padding: 8px;
  margin: 5px 0;
  width: 100%;
}

Additional Considerations:

    API Access: If you are dealing with large-scale scraping, it might be better to use the Facebook Marketing API for legal and more stable access to data.
    Throttling and Respecting TOS: Be cautious with scraping to avoid being blocked or violating Facebook's terms of service.
    Data Export: Once ads are scraped, consider allowing the user to export data to CSV or store it for later analysis.

Conclusion

This Chrome extension allows you to interact with the Facebook Ads Library to filter ads by keyword, country, and timeframe, then extract key information about the ads. You can add more filters and extend its functionality to suit your needs. Make sure to comply with Facebook's terms of service while using or distributing the tool.
