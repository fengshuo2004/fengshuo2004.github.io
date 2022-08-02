---
layout: post
title: Scraping and downloading photos from eStream
---

I've made a script that extracts photo URLs from Planet eStream and another script to download those photos in batch.

## To get a URL list of photos

1. Download Tampermonkey browser extension if you don't have it already.

2. Click its icon, then **+ Create a new script...**

3. Copy and paste in the content of *estream.user.js* then save the script.

4. Go on eStream photo library.

5. Click to display all photos to get all photos; or click the specific collection you wish to have.

6. Make sure the browser window is in focus, press `F12` to open Developer Tools then switch to the Console tab.

7. Execute `scrapeStart();`

8. Wait for the scraping to finish. A text file containing direct links to photos will be downloaded at the end.

## Listing *estream.user.js*

```javascript
// ==UserScript==
// @name         Planet eStream Photo Library Scraper
// @namespace    io.github.fengshuo2004
// @version      0.1
// @description  Extracts photo URLs from Planet eStream for you to download in batch
// @author       David Feng
// @match        https://estream.shrewsbury.org.uk/PL/Default.aspx*
// @icon         https://estream.shrewsbury.org.uk/favicon.ico
// @grant        none
// @require      http://danml.com/js/download2.js
// ==/UserScript==

(function () {
    'use strict';

    // Delay in milliseconds between subsequent clicks
    const delayMs = 1000; // <-- CHANGE THIS

    // Where all the URL are stored
    let photoURLs = [];

    // Open console and type scrapeStart() to begin
    window.scrapeStart = function(){
        // Simulate a click on the first photo
        document.querySelector("div.search__resultitem:nth-child(1)").click();
        let intervalID = setInterval(function(){
            // Has the "End of results" box popped up?
            if (document.querySelector(".iziToast-capsule") != null) {
                window.scrapeAbort(intervalID);
            }
            let imgSrc = document.querySelector("#quickview-image").getAttribute("src");
            if (imgSrc != null){
                // Extract the src attribute of the img tag
                photoURLs.push(imgSrc);
                // Simulate a click on the ">" button
                document.querySelector("#btnPV9").click();
            }
        }, delayMs);
        console.log("Scraping started, execute scrapeAbort(" + intervalID.toString() + ") to stop manually");
    }

    // Call this to abort
    window.scrapeAbort = function(itvID){
        clearInterval(itvID);
        console.log("We've reached the end. There are " + photoURLs.length + " photos in total.");
        // Format the URLs into a string
        let URLstring = "";
        for (const i of photoURLs){
            URLstring += window.location.origin + i.split("?")[0] + "\n"
        }
        // Save it
        let fileName;
        if (document.querySelector("#lbl_CollectionTitle")){ // Is a collection?
            // Extract collection title from element
            let temp = document.querySelector("#lbl_CollectionTitle").innerText;
            // Sanitise: remove special chars, replace space with underscore
            temp = temp.replace(/[`!@#$%&*|+=?:'".<>\{\}\\\/]/g, "");
            temp = temp.replaceAll(" ", "_");
            fileName = temp + ".txt";
        } else { // Scraped the entire library, use fallback name
            fileName = "photo_urls.txt";
        }
        download(URLstring, fileName, "text/plain");
        console.log("Saved a list of URLs to your downloads folder. Thank you come again!")
    }
})();
```

## To download the photos in batch

Once you have the text file, you can use any download manager software that supports batch download (one URL per line), for example IDM or Motrix. Alternatively you can use the python script below to automate it.

## Listing *dl.py*

```python
import os, sys, requests

def main():
    if len(sys.argv) == 3:
        with open(sys.argv[1], 'r') as f:
            lines = f.read().splitlines() 
        try:
            os.makedirs(sys.argv[2])
        except FileExistsError:
            pass
        length = len(lines)
        for i, u in enumerate(lines):
            file = requests.get(u, allow_redirects=True)
            filename = u.rsplit('/', 1)[1]
            with open(sys.argv[2] + "/" + filename, "wb") as f:
                f.write(file.content)
            print(str((i + 1) / length * 100)[0:5] + "% | " + filename)
        print("Done")
    else:
        print("Usage:")
        print("python dl.py <file with URLs> <download destination>")

if __name__ == "__main__":
    main()
```
