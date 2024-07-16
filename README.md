<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple API Tester</title>
    <style>
        body {
            font-family: Arial, sans-serif;
        }
        .container {
            width: 80%;
            margin: auto;
            padding: 20px;
            border: 1px solid #ccc;
            border-radius: 10px;
            background-color: #f9f9f9;
        }
        .input-group {
            margin-bottom: 20px;
        }
        .input-group label {
            display: block;
            margin-bottom: 5px;
        }
        .input-group input,
        .input-group select,
        .input-group textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
        .input-group textarea {
            resize: vertical;
        }
        button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            background-color: #007bff;
            color: #fff;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
        .response {
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Simple API Tester</h1>
        <div class="input-group">
            <label for="method">Method</label>
            <select id="method">
                <option value="GET">GET</option>
                <option value="POST">POST</option>
                <option value="PATCH">PATCH</option>
                <option value="PUT">PUT</option>
                <option value="DELETE">DELETE</option>
            </select>
        </div>
        <div class="input-group">
            <label for="url">URL</label>
            <input type="text" id="url" placeholder="Enter request URL">
        </div>
        <div class="input-group">
            <label for="headers">Headers</label>
            <textarea id="headers" rows="4" placeholder='key1:value1\nkey2:value2'></textarea>
        </div>
        <div class="input-group">
            <label for="body">Body (JSON format)</label>
            <textarea id="body" rows="6" placeholder='{"key": "value"}'></textarea>
        </div>
        <button onclick="sendRequest()">Send Request</button>
        <button onclick="sendCurl()">Send cURL</button>
        <div class="input-group">
            <label for="curl">cURL Command</label>
            <textarea id="curl" rows="4" placeholder="Enter cURL command here"></textarea>
        </div>
        <div class="response">
            <h3>Response</h3>
            <pre id="response"></pre>
        </div>
    </div>

    <script>
        async function sendRequest() {
            const method = document.getElementById('method').value;
            const url = document.getElementById('url').value;
            const headersText = document.getElementById('headers').value;
            const body = document.getElementById('body').value;

            let headers = {};
            headersText.split('\n').forEach(header => {
                const [key, value] = header.split(':');
                if (key && value) {
                    headers[key.trim()] = value.trim();
                }
            });

            const options = {
                method: method,
                headers: headers,
                body: ['GET', 'DELETE'].includes(method) ? null : body
            };

            try {
                const response = await fetch(url, options);
                const responseData = await response.text();
                document.getElementById('response').innerText = responseData;
            } catch (error) {
                document.getElementById('response').innerText = 'Error: ' + error;
            }
        }

        async function sendCurl() {
            const curlCommand = document.getElementById('curl').value;
            const curlParts = curlCommand.match(/(?:'[^']*'|"[^"]*"|[^'"\s])+/g);

            let method = 'GET';
            let url = '';
            let headers = {};
            let body = '';

            for (let i = 0; i < curlParts.length; i++) {
                const part = curlParts[i];

                if (part === '-X' || part === '--request') {
                    method = curlParts[++i];
                } else if (part === '-H' || part === '--header') {
                    const header = curlParts[++i].replace(/(^"|"$)/g, '').split(': ');
                    headers[header[0]] = header[1];
                } else if (part === '-d' || part === '--data') {
                    body = curlParts[++i].replace(/(^'|'$)/g, '');
                } else if (!part.startsWith('-')) {
                    url = part;
                }
            }

            const options = {
                method: method,
                headers: headers,
                body: ['GET', 'DELETE'].includes(method) ? null : body
            };

            try {
                const response = await fetch(url, options);
                const responseData = await response.text();
                document.getElementById('response').innerText = responseData;
            } catch (error) {
                document.getElementById('response').innerText = 'Error: ' + error;
            }
        }
    </script>
</body>
</html>
