---
layout: default
title: SRE Tools
categories: [Tools]
permalink: /hostname/
---

<div id="page-content">
            <h1>SRE Swiss army knife - Hostname Extractor and Formatter</h1>
            <h3>This is a simple Hostname Extractor and Formatter that will format valid FQDNs into command line readable formats in bulk operation</h3>
    Text filter regex pattern: <input id="inputPattern" size=50 value="\b[a-z0-9-]+(?:\.[a-z0-9-]+)+\.[a-z]{2,}\b">

    <br><br>
    <textarea id="inputText" rows="18" cols="85" placeholder="Paste your servername/fqdn here alongside a bunch of random text and click the format buttons below, you will find that the app names are filtered and formatted to be command line friendly.">Paste your servername/fqdn here alongside a bunch of random text and click the format buttons below, you will find that the app names are filtered and formatted to be command line/terminal friendly. 
The default REGEX is designed to filter the sample hostnames below but you can change it to whatever suits you. 

See example here: https://regex101.com/r/TITW2V/1

------
server name: datacenter-server117626.prod.example.com. CPU idle 98%
server name: datacenter5.staging.example.com  CPU idle 65%
------
application-name  status
webapp100.dc2.com UP
webapp300.dc2.com UP

</textarea>
    <br>
    <button onclick="extractHostnames()">Format to comma separated</button>
    <button onclick="formatHostnames()">Format to newline separated</button>

    <p><i> Privacy note: Your data is NOT transmitted over the network. All operations happen locally on the browser only. </i></p>

    <h2>Comma-Separated Hostnames:</h2>
    <p id="outputText"></p>
    <h2>Newline-Separated Hostnames:</h2>
    <div id="formattedText"></div>

    <script>
         //author: github.com/tgangte -  2024 
        function extractHostnames() {
            // Get input text
            const inputText = document.getElementById('inputText').value;
            const patternText = document.getElementById('inputPattern').value;

            const hostnamePattern = new RegExp(patternText, 'g');
            const matches = inputText.match(hostnamePattern);
            // Convert matches to a comma-separated string
            const commaSeparatedHostnames = matches ? matches.join(',') : '';

            document.getElementById('outputText').textContent = commaSeparatedHostnames;
        }

        function formatHostnames() {
            // Get input text
            const inputText = document.getElementById('inputText').value;
            const patternText = document.getElementById('inputPattern').value;
            const hostnamePattern = new RegExp(patternText, 'g');
 
            // Extract all matches
            const matches = inputText.match(hostnamePattern);

            // Convert matches to a newline-separated strings with <br> tags
            const newlineSeparatedHostnames = matches ? matches.join('<br>') : '';

            document.getElementById('formattedText').innerHTML = newlineSeparatedHostnames;
        }
    </script>
      
</div>
