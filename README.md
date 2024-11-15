
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
            max-height: 300px;
            overflow-y: scroll;
            border: 1px solid #ddd;
            padding: 10px;
            border-radius: 4px;
            background: #f9f9f9;
        }
        .bill-item {
            background: #ffffff;
            padding: 10px;
            margin: 5px 0;
            border-radius: 4px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            color: red;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        }
        .bill-item.paid {
            color: green;
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
        .bill-item button.edit {
            background-color: #ffc107;
        }
        .total-amount {
            margin-top: 20px;
            text-align: center;
            font-size: 18px;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div class="container">
        <form id="bill-form">
            <input type="text" id="bill-name" placeholder="Bill Name" required>
            <input type="number" id="bill-amount" placeholder="Amount" required step="0.01">
            <input type="date" id="bill-due-date" required>
            <button type="submit">Add Bill</button>
        </form>
        <div class="bill-list" id="bill-list"></div>
        <div class="total-amount" id="total-amount">Total: £0.00</div>
    </div>

    <script>
        let editingIndex = null; // Track which bill is being edited

        document.getElementById('bill-form').addEventListener('submit', function(event) {
            event.preventDefault();
            
            const billName = document.getElementById('bill-name').value;
            const billAmount = parseFloat(document.getElementById('bill-amount').value);
            const billDueDate = document.getElementById('bill-due-date').value;

            if (isNaN(billAmount)) {
                alert("Please enter a valid amount.");
                return;
            }

            if (editingIndex !== null) {
                updateBill(editingIndex, billName, billAmount, billDueDate);
            } else {
                addBill(billName, billAmount, billDueDate);
            }

            document.getElementById('bill-form').reset();
            editingIndex = null; // Reset editing index after saving
        });

        function addBill(name, amount, dueDate) {
            const billItem = {
                name,
                amount,
                dueDate,
                paid: false
            };

            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.push(billItem);
            bills.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate)); // Sort by due date
            localStorage.setItem('bills', JSON.stringify(bills));

            displayBills();
        }

        function updateBill(index, name, amount, dueDate) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills[index] = { ...bills[index], name, amount, dueDate }; // Update the bill details
            bills.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate)); // Re-sort by due date
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
                    <span>${bill.name} - £${parseFloat(bill.amount).toFixed(2)} - Due: ${bill.dueDate}</span>
                    <button onclick="markAsPaid(${index})" class="${bill.paid ? 'paid' : ''}" ${bill.paid ? 'disabled' : ''}>${bill.paid ? 'Paid' : 'Mark as Paid'}</button>
                    <button class="edit" onclick="editBill(${index})">Edit</button>
                    <button class="remove" onclick="removeBill(${index})">Remove</button>
                `;
                billList.appendChild(billItem);
            });

            calculateTotal();
        }

        function editBill(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            const bill = bills[index];

            // Populate form fields with existing bill details
            document.getElementById('bill-name').value = bill.name;
            document.getElementById('bill-amount').value = bill.amount;
            document.getElementById('bill-due-date').value = bill.dueDate;

            editingIndex = index; // Set the editing index to save changes
        }

        function markAsPaid(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            
            const bill = bills[index];
            bill.paid = true;

            const nextMonthBill = {
                ...bill,
                paid: false, // Reset paid status for the new bill
                dueDate: getNextMonthDate(bill.dueDate) // Shift due date to next month
            };

            bills.push(nextMonthBill); // Add the next month's bill
            localStorage.setItem('bills', JSON.stringify(bills));

            displayBills();
        }

        function getNextMonthDate(currentDate) {
            const date = new Date(currentDate);
            date.setMonth(date.getMonth() + 1);
            if (date.getDate() !== parseInt(currentDate.split('-')[2])) {
                date.setDate(0); // Handle shorter months
            }
            return date.toISOString().split('T')[0];
        }

        function removeBill(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.splice(index, 1);
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        function calculateTotal() {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            const total = bills.reduce((sum, bill) => sum + (bill.paid ? 0 : bill.amount), 0);
            document.getElementById('total-amount').textContent = `Total: £${total.toFixed(2)}`;
        }

        // Display bills on page load
        displayBills();
    </script>
</body>
</html>