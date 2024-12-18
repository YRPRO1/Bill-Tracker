
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Budget Tracker</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f4f4f4;
            overflow-y: scroll;
        }
        .container {
            width: 350px;
            background: white;
            padding: 20px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
            margin: 20px auto;
        }
        h2 {
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
        .checkbox-group {
            display: flex;
            flex-direction: column;
            margin-top: 10px;
        }
        .checkbox-group label {
            font-size: 16px;
            margin: 5px 0;
        }
        .bill-list, .money-in-list {
            margin-top: 20px;
            max-height: 300px;
            overflow-y: scroll;
            border: 1px solid #ddd;
            padding: 10px;
            border-radius: 4px;
            background: #f9f9f9;
        }
        .bill-item, .income-item {
            background: #ffffff;
            padding: 10px;
            margin: 5px 0;
            border-radius: 4px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            flex-wrap: wrap;
        }
        .bill-item span.amount {
            color: red;
        }
        .bill-item.paid span.amount {
            color: green;
        }
        .bill-item.paid {
            background-color: #e0ffe0;
        }
        .total-amount {
            margin-top: 20px;
            text-align: center;
            font-size: 18px;
            font-weight: bold;
        }
        .balance-section {
            margin-top: 20px;
            padding: 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            background: #f9f9f9;
        }
        .balance-section span {
            font-size: 18px;
            font-weight: bold;
        }
        .action-buttons {
            display: flex;
            gap: 5px;
            margin-top: 5px;
        }
        .action-buttons button {
            font-size: 12px;
            padding: 5px;
            background-color: #28a745;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        .action-buttons .remove {
            background-color: #dc3545;
        }
        .action-buttons .paid {
            background-color: #007bff;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- Bill Form -->
        <form id="bill-form">
            <h2>Budget Tracker</h2>
            <h3>Add Bill</h3>
            <input type="text" id="bill-name" placeholder="Bill Name" required>
            <input type="number" id="bill-amount" placeholder="Amount" required step="0.01">
            <input type="date" id="bill-due-date" required>

            <!-- Checkboxes for Payment Type -->
            <div class="checkbox-group">
                <label><input type="checkbox" id="one-off" onclick="toggleCheckbox('one-off')"> One-off Payment</label>
                <label><input type="checkbox" id="recurring" onclick="toggleCheckbox('recurring')"> Recurring Payment</label>
            </div>

            <button type="submit">Add Bill</button>
        </form>

        <div class="bill-list" id="bill-list"></div>
        <div class="total-amount" id="total-amount">Total Bills: £0.00</div>

        <!-- Income Form -->
        <form id="income-form">
            <h3>Add Income</h3>
            <input type="text" id="income-name" placeholder="Income Source" required>
            <input type="number" id="income-amount" placeholder="Amount" required step="0.01">
            <button type="submit">Add Income</button>
        </form>

        <div class="money-in-list" id="money-in-list"></div>
        <div class="total-amount" id="total-income">Total Income: £0.00</div>

        <!-- Balance Section -->
        <div class="balance-section" id="balance-section">
            Remaining Balance: <span id="remaining-balance">£0.00</span>
        </div>
    </div>

    <script>
        let editingIndex = null;

        function toggleCheckbox(id) {
            if (id === 'one-off') {
                document.getElementById('recurring').checked = false;
            } else {
                document.getElementById('one-off').checked = false;
            }
        }

        // Bills Management
        document.getElementById('bill-form').addEventListener('submit', function(event) {
            event.preventDefault();
            const billName = document.getElementById('bill-name').value;
            const billAmount = parseFloat(document.getElementById('bill-amount').value);
            const billDueDate = document.getElementById('bill-due-date').value;
            const isOneOff = document.getElementById('one-off').checked;
            const isRecurring = document.getElementById('recurring').checked;

            if (isNaN(billAmount)) {
                alert("Please enter a valid amount.");
                return;
            }

            addBill(billName, billAmount, billDueDate, isOneOff, isRecurring);
            document.getElementById('bill-form').reset();
        });

        function addBill(name, amount, dueDate, oneOff, recurring) {
            const billItem = { name, amount, dueDate, paid: false, oneOff, recurring };
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.push(billItem);
            bills.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate)); // Sort bills by date
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
                    <span>${bill.name}</span>
                    <span class="amount">£${bill.amount.toFixed(2)}</span>
                    <span>${bill.dueDate}</span>
                    <div class="action-buttons">
                        <button onclick="markAsPaid(${index})" class="paid">${bill.paid ? 'Paid' : 'Mark as Paid'}</button>
                        <button onclick="editBill(${index})" class="edit">Edit</button>
                        <button onclick="removeBill(${index})" class="remove">Remove</button>
                    </div>
                `;
                billList.appendChild(billItem);
            });
            calculateTotals();
        }

        function markAsPaid(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            const bill = bills[index];

            // Mark the bill as paid and turn it green
            bill.paid = true;

            if (bill.oneOff) {
                // Remove the one-off bill once marked as paid
                bills.splice(index, 1);
            } else if (bill.recurring) {
                // For recurring bills, mark as paid and set a flag to add the next bill only in the new month
                const nextMonth = getNextMonthDate(bill.dueDate);
                bills[index] = {
                    ...bill,
                    dueDate: nextMonth,
                    paid: true,
                    nextDue: nextMonth,  // Mark next due date but only add in the new month
                };
            }

            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        function getNextMonthDate(currentDate) {
            const date = new Date(currentDate);
            date.setMonth(date.getMonth() + 1);
            if (date.getDate() !== parseInt(currentDate.split('-')[2])) {
                date.setDate(0);
            }
            return date.toISOString().split('T')[0];
        }

        function editBill(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            const bill = bills[index];

            document.getElementById('bill-name').value = bill.name;
            document.getElementById('bill-amount').value = bill.amount;
            document.getElementById('bill-due-date').value = bill.dueDate;
            document.getElementById('one-off').checked = bill.oneOff;
            document.getElementById('recurring').checked = bill.recurring;

            editingIndex = index;
        }

        function removeBill(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.splice(index, 1);
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        // Income Management
        document.getElementById('income-form').addEventListener('submit', function(event) {
            event.preventDefault();
            const incomeName = document.getElementById('income-name').value;
            const incomeAmount = parseFloat(document.getElementById('income-amount').value);

            if (isNaN(incomeAmount)) {
                alert("Please enter a valid amount.");
                return;
            }

            addIncome(incomeName, incomeAmount);
            document.getElementById('income-form').reset();
        });

        function addIncome(name, amount) {
            const incomeItem = { name, amount };
            let incomes = JSON.parse(localStorage.getItem('incomes')) || [];
            incomes.push(incomeItem);
            localStorage.setItem('incomes', JSON.stringify(incomes));
            displayIncomes();
        }

        function displayIncomes() {
            const moneyInList = document.getElementById('money-in-list');
            moneyInList.innerHTML = '';
            let incomes = JSON.parse(localStorage.getItem('incomes')) || [];
            incomes.forEach((income, index) => {
                const incomeItem = document.createElement('div');
                incomeItem.className = 'income-item';
                incomeItem.innerHTML = `
                    <span>${income.name} - £${income.amount.toFixed(2)}</span>
                    <button onclick="removeIncome(${index})">Remove</button>
                `;
                moneyInList.appendChild(incomeItem);
            });
            calculateTotals();
        }

        function removeIncome(index) {
            let incomes = JSON.parse(localStorage.getItem('incomes')) || [];
            incomes.splice(index, 1);
            localStorage.setItem('incomes', JSON.stringify(incomes));
            displayIncomes();
        }

        // Calculate Totals and Remaining Balance
        function calculateTotals() {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            let incomes = JSON.parse(localStorage.getItem('incomes')) || [];

            const totalBills = bills.reduce((sum, bill) => sum + (!bill.paid ? bill.amount : 0), 0);
            const totalIncome = incomes.reduce((sum, income) => sum + income.amount, 0);

            document.getElementById('total-amount').textContent = `Total Bills: £${totalBills.toFixed(2)}`;
            document.getElementById('total-income').textContent = `Total Income: £${totalIncome.toFixed(2)}`;
            document.getElementById('remaining-balance').textContent = `£${(totalIncome - totalBills).toFixed(2)}`;
        }

        // Initialize
        displayBills();
        displayIncomes();
    </script>
</body>
</html>