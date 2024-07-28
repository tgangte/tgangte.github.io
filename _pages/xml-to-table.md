---
layout: default
title: SRE Tools - xml to table converter
categories: [Tools]
permalink: /xml-to-table/
---
<div id="page-content">
            <h1>SRE Swiss army knife - XML to Table converter </h1>
            <h4>This tool will convert any XML code into a HTML table that is pretty and easily human readable</h4>
</div>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>XML to HTML Table Converter</title>
    <style>
        table {
            border-collapse: collapse;
            width: 100%;
            margin-bottom: 20px;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
        .nested-table {
            margin-left: 20px;
        }
        .attribute {
            color: #0066cc;
            font-style: italic;
        }
    </style>
</head>
<body>
    <textarea id="xmlInput" rows="18" cols="85" placeholder="Paste your valid XML text here and click the convert button"><bookstore>  
  <book category="COOKING">  
    <title lang="en">Everyday Italian</title>  
    <author>Giada De Laurentiis</author>  
    <year>2005</year>  
    <price>30.00</price>  
  </book>  
  <book category="COOKING">  
    <title lang="en">Everyday Gibberish</title>  
    <author>Bumbleee </author>  
    <year>2025</year>  
    <price>77.00</price>  
  </book>  
  <book category="CHILDREN">  
    <title lang="en">Harry Potter</title>  
    <author>J K. Rowling</author>  
    <year>2005</year>  
    <price>29.99</price>  
  </book>  
  <book category="WEB">  
    <title lang="en">Learning XML</title>  
    <author>Erik T. Ray</author>  
    <year>2003</year>  
    <price>39.95</price>  
  </book>  
</bookstore>  </textarea>
    <br>
    <button onclick="convertXmlToTable()">Convert to Table</button>
    <p><i> Privacy note: Your data is NOT transmitted over the network. All operations happen locally on the browser only. </i></p>

    <div id="tableOutput"></div>

    <script>
        function convertXmlToTable() {
            const xmlString = document.getElementById('xmlInput').value;
            const parser = new DOMParser();
            const xmlDoc = parser.parseFromString(xmlString, 'text/xml');
            
            function createTable(node) {
                let table = '<table class="nested-table">';
                table += '<tr><th>Element</th><th>Value</th></tr>';

                for (let child of node.childNodes) {
                    if (child.nodeType === Node.ELEMENT_NODE) {
                        let elementName = child.nodeName;
                        let attributes = Array.from(child.attributes).map(attr => 
                            `<span class="attribute">${attr.name}="${attr.value}"</span>`
                        ).join(' ');
                        
                        if (attributes) {
                            elementName += ' ' + attributes;
                        }

                        table += `<tr><td>${elementName}</td><td>`;
                        if (child.childNodes.length === 1 && child.childNodes[0].nodeType === Node.TEXT_NODE) {
                            table += child.textContent.trim();
                        } else if (child.childNodes.length > 0) {
                            table += createTable(child);
                        }
                        table += '</td></tr>';
                    }
                }

                table += '</table>';
                return table;
            }

            const table = createTable(xmlDoc.documentElement);
            document.getElementById('tableOutput').innerHTML = table;
        }
    </script>
</body>
