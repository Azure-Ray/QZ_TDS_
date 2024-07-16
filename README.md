<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple API Tester</title>
</head>
<body style="font-family: Arial, sans-serif;">
    <div class="container" style="width: 80%; margin: auto; padding: 20px; border: 1px solid #ccc; border-radius: 10px; background-color: #f9f9f9;">
        <h1>Simple API Tester</h1>
        <div class="input-group" style="margin-bottom: 20px;">
            <label for="method" style="display: block; margin-bottom: 5px;">Method</label>
            <select id="method" style="width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px;">
                <option value="GET">GET</option>
                <option value="POST">POST</option>
                <option value="PATCH">PATCH</option>
                <option value="PUT">PUT</option>
                <option value="DELETE">DELETE</option>
            </select>
        </div>
        <div class="input-group" style="margin-bottom: 20px;">
            <label for="url" style="display: block; margin-bottom: 5px;">URL</label>
            <input type="text" id="url" placeholder="Enter request URL" style="width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px;">
        </div>
        <div class="input-group" style="margin-bottom: 20px;">
            <label for="headers" style="display: block; margin-bottom: 5px;">Headers</label>
            <textarea id="headers" rows="4" placeholder='key1:value1\nkey2:value2' style="width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; resize: vertical;"></textarea>
        </div>
        <div class="input-group" style="margin-bottom: 20px;">
            <label for="body" style="display: block; margin-bottom: 5px;">Body (JSON format)</label>
            <textarea id="body" rows="6" placeholder='{"key": "value"}' style="width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; resize: vertical;"></textarea>
        </div>
        <button onclick="sendRequest()" style="padding: 10px 20px; border: none; border-radius: 5px; background-color: #007bff; color: #fff; cursor: pointer;">Send Request</button>
        <button onclick="sendCurl()" style="padding: 10px 20px; border: none; border-radius: 5px; background-color: #007bff; color: #fff; cursor: pointer; margin-left: 10px;">Send cURL</button>
        <div class="input-group" style="margin-top: 20px;">
            <label for="curl" style="display: block; margin-bottom: 5px;">cURL Command</label>
            <textarea id="curl" rows="4" placeholder="Enter cURL command here" style="width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; resize: vertical;"></textarea>
        </div>
        <div class="response" style="margin-top: 20px;">
            <h3>Response</h3>
            <pre id="response" style="white-space: pre-wrap;"></pre>
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
