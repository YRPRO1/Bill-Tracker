
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
        }
        .bill-item span.amount {
            color: red;
        }
        .bill-item.paid span.amount {
            color: green;
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
    </style>
</head>
<body>
    <div class="container">
        <!-- Bill Form -->
        <form id="bill-form">
            <h2>Budget Tracker</h2>
            <h3>Add/Edit Bill</h3>
            <input type="text" id="bill-name" placeholder="Bill Name" required>
            <input type="number" id="bill-amount" placeholder="Amount" required step="0.01">
            <input type="date" id="bill-due-date" required>
            <button type="submit">Add Bill</button>
        </form>

        <div class="bill-list" id="bill-list"></div>
        <div class="total-amount" id="total-amount">Total Bills: £0.00</div>
    </div>

    <script>
        let editingIndex = null;

        // Bills Management
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
            editingIndex = null;
        });

        function addBill(name, amount, dueDate) {
            const billItem = { name, amount, dueDate, paid: false };
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.push(billItem);
            bills.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate)); // Sort by due date
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        function updateBill(index, name, amount, dueDate) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills[index] = { name, amount, dueDate, paid: bills[index].paid }; // Retain 'paid' status
            bills.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate)); // Re-sort after editing
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
                    <button onclick="markAsPaid(${index})" ${bill.paid ? 'disabled' : ''}>${bill.paid ? 'Paid' : 'Mark as Paid'}</button>
                    <button onclick="editBill(${index})">Edit</button>
                    <button onclick="removeBill(${index})">Remove</button>
                `;
                billList.appendChild(billItem);
            });
            calculateTotals();
        }

        function markAsPaid(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills[index].paid = true;
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        function editBill(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            const bill = bills[index];

            document.getElementById('bill-name').value = bill.name;
            document.getElementById('bill-amount').value = bill.amount;
            document.getElementById('bill-due-date').value = bill.dueDate;

            editingIndex = index;
        }

        function removeBill(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.splice(index, 1);
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }
    </script>
</body>
    <script>
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

        // Totals and Balance Calculation
        function calculateTotals() {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            let incomes = JSON.parse(localStorage.getItem('incomes')) || [];

            const totalBills = bills.reduce((sum, bill) => sum + (!bill.paid ? bill.amount : 0), 0);
            const totalIncome = incomes.reduce((