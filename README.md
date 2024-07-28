# Bill-Tracker
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Bill Tracker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f4f4f4;
            margin: 0;
        }
        .container {
            width: 300px;
            background: white;
            padding: 20px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
        }
        h1 {
            text-align: center;
        }
        form {
            display: flex;
            flex-direction: column;
        }
        input, button {
            padding: 10px;
            margin: 5px 0;
        }
        button {
            background-color: #28a745;
            color: white;
            border: none;
            cursor: pointer;
        }
        .bill-list {
            margin-top: 20px;
        }
        .bill-item {
            background: #f9f9f9;
            padding: 10px;
            margin: 5px 0;
            border-radius: 4px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: red; /* Default to red */
        }
        .bill-item.paid {
            color: green; /* Change to green when paid */
        }
        .bill-item span {
            flex-grow: 1;
        }
        .bill-item button {
            background-color: #007bff;
            color: white;
            border: none;
            padding: 5px;
            cursor: pointer;
            border-radius: 4px;
            margin-left: 5px;
        }
        .bill-item button.paid {
            background-color: #28a745;
        }
        .bill-item button.remove {
            background-color: #dc3545;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Bill Tracker</h1>
        <form id="bill-form">
            <input type="text" id="bill-name" placeholder="Bill Name" required>
            <input type="text" id="bill-amount" placeholder="Amount" required>
            <input type="date" id="bill-due-date" required>
            <button type="submit">Add Bill</button>
        </form>
        <div class="bill-list" id="bill-list"></div>
    </div>

    <script>
        document.getElementById('bill-form').addEventListener('submit', function(event) {
            event.preventDefault();
            
            const billName = document.getElementById('bill-name').value;
            const billAmount = document.getElementById('bill-amount').value;
            const billDueDate = document.getElementById('bill-due-date').value;

            addBill(billName, billAmount, billDueDate);

            // Clear the form
            document.getElementById('bill-form').reset();
        });

        function addBill(name, amount, dueDate) {
            const billItem = {
                name: name,
                amount: amount,
                dueDate: dueDate,
                paid: false
            };

            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.push(billItem);
            bills.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate));
            localStorage.setItem('bills', JSON.stringify(bills));

            displayBills();
        }

        function displayBills() {
            const billList = document.getElementById('bill-list');
            billList.innerHTML = '';
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.forEach((bill, index) => {
                const billItem = document.createElement('div');
                billItem.className = 'bill-item' + (bill.paid ? ' paid' : '');
                billItem.innerHTML = `
                    <span>${bill.name} - ${isNaN(bill.amount) ? bill.amount : '£' + parseFloat(bill.amount).toFixed(2)} - Due: ${bill.dueDate}</span>
                    <button onclick="markAsPaid(${index})" class="${bill.paid ? 'paid' : ''}" ${bill.paid ? 'disabled' : ''}>${bill.paid ? 'Paid' : 'Mark as Paid'}</button>
                    <button class="remove" onclick="removeBill(${index})">Remove</button>
                `;
                billList.appendChild(billItem);
            });
        }

        function markAsPaid(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills[index].paid = true;
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        function removeBill(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.splice(index, 1);
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        function resetBills() {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills = bills.map(bill => ({ ...bill, paid: false }));
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        // Check if it's a new month and reset bills
        const currentMonth = new Date().getMonth();
        if (localStorage.getItem('lastMonth') != currentMonth) {
            localStorage.setItem('lastMonth', currentMonth);
            resetBills();
        }

        // Display bills on page load
        displayBills();
    </script>
</body>
</html>
