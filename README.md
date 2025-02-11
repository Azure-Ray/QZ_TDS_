<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>API List</title>
</head>
<body style="font-family: Arial, sans-serif; background-color: #f4f6f8; margin: 0; padding: 0;">
    <div style="width: 90%; margin: 20px auto; background-color: #ffffff; padding: 20px; border-radius: 8px; box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);">
        <h1 style="color: #005288; font-size: 24px; text-align: center;">API List Management</h1>
        <form id="addForm" style="display: flex; justify-content: space-between; margin-bottom: 20px;">
            <input type="text" id="apiLink" placeholder="API Link" required style="padding: 8px; width: 30%; margin-right: 10px; border: 1px solid #ddd; border-radius: 4px;">
            <input type="email" id="mailContact" placeholder="Mail Contact Point" required style="padding: 8px; width: 30%; margin-right: 10px; border: 1px solid #ddd; border-radius: 4px;">
            <input type="text" id="lastMaintainer" placeholder="Last Maintainer (English)" required style="padding: 8px; width: 30%; margin-right: 10px; border: 1px solid #ddd; border-radius: 4px;">
            <button type="submit" class="button" style="background-color: #005288; color: white; padding: 8px 12px; border: none; border-radius: 4px; cursor: pointer; font-size: 14px;">Add API</button>
        </form>
        
        <table id="apiTable" style="width: 100%; border-collapse: collapse; margin-top: 20px;">
            <thead>
                <tr style="background-color: #005288; color: white; font-weight: bold;">
                    <th style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">API Link</th>
                    <th style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">Mail Contact Point</th>
                    <th style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">Last Maintainer</th>
                    <th style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">Actions</th>
                </tr>
            </thead>
            <tbody style="background-color: #f9f9f9;">
                <!-- Rows will be inserted dynamically -->
            </tbody>
        </table>

        <div class="pagination" style="display: flex; justify-content: center; margin-top: 20px;">
            <button id="prevPage" style="background-color: #f4f6f8; border: 1px solid #ddd; color: #005288; padding: 8px 12px; cursor: pointer; margin: 0 5px;">Previous</button>
            <button id="nextPage" style="background-color: #f4f6f8; border: 1px solid #ddd; color: #005288; padding: 8px 12px; cursor: pointer; margin: 0 5px;">Next</button>
        </div>
    </div>

    <script>
        let currentPage = 1;
        const itemsPerPage = 5;

        async function fetchData(page) {
            try {
                const response = await fetch(`/api/getApis?page=${page}&size=${itemsPerPage}`);
                const data = await response.json();
                return data;
            } catch (error) {
                console.error("Error fetching data:", error);
            }
        }

        async function updateTable() {
            const tableBody = document.querySelector("#apiTable tbody");
            tableBody.innerHTML = '';
            const apiData = await fetchData(currentPage);

            if (apiData) {
                apiData.forEach(api => {
                    const row = document.createElement("tr");
                    row.innerHTML = `
                        <td style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">${api.link}</td>
                        <td style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">${api.contact}</td>
                        <td style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">${api.maintainer}</td>
                        <td style="padding: 12px 15px; text-align: left; border-bottom: 1px solid #ddd;">
                            <button onclick="editApi(${api.id})" style="background-color: #005288; color: white; padding: 8px 12px; border: none; border-radius: 4px; cursor: pointer;">Edit</button>
                        </td>
                    `;
                    tableBody.appendChild(row);
                });
            }

            document.querySelector("#prevPage").disabled = currentPage === 1;
            document.querySelector("#nextPage").disabled = apiData.length < itemsPerPage;
        }

        async function addApi(event) {
            event.preventDefault();
            const apiLink = document.querySelector("#apiLink").value;
            const mailContact = document.querySelector("#mailContact").value;
            const lastMaintainer = document.querySelector("#lastMaintainer").value;

            const response = await fetch('/api/addApi', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ link: apiLink, contact: mailContact, maintainer: lastMaintainer }),
            });

            if (response.ok) {
                updateTable();
                event.target.reset();
            } else {
                alert("Failed to add API.");
            }
        }

        function editApi(id) {
            const newLink = prompt("Edit API Link");
            const newContact = prompt("Edit Mail Contact Point");
            const newMaintainer = prompt("Edit Last Maintainer");

            if (newLink && newContact && newMaintainer) {
                fetch(`/api/editApi/${id}`, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ link: newLink, contact: newContact, maintainer: newMaintainer }),
                }).then(() => updateTable());
            }
        }

        document.querySelector("#addForm").addEventListener("submit", addApi);
        document.querySelector("#prevPage").addEventListener("click", () => {
            if (currentPage > 1) {
                currentPage--;
                updateTable();
            }
        });
        document.querySelector("#nextPage").addEventListener("click", () => {
            currentPage++;
            updateTable();
        });

        updateTable(); // Initial load
    </script>
</body>
</html>
