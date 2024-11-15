
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
            overflow-y: scroll; /* Enable vertical scrolling */
        }
        .container {
            width: 350px;
            background: white;
            padding: 20px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
            margin: 20px auto; /* Center the container */
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
        <!-- Form for Adding Bills -->
        <form id="bill-form">
            <h2>Budget Tracker</h2>
            <h3>Add Bill</h3>
            <input type="text" id="bill-name" placeholder="Bill Name" required>
            <input type="number" id="bill-amount" placeholder="Amount" required step="0.01">
            <input type="date" id="bill-due-date" required>
            <button type="submit">Add Bill</button>
        </form>

        <div class="bill-list" id="bill-list"></div>
        <div class="total-amount" id="total-amount">Total Bills: £0.00</div>

        <!-- Form for Adding Income -->
        <form id="income-form">
            <h3>Add Monthly Income</h3>
            <input type="text" id="income-name" placeholder="Income Source" required>
            <input type="number" id="income-amount" placeholder="Amount" required step="0.01">
            <input type="month" id="income-month" required>
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
        document.getElementById('bill-form').addEventListener('submit', function(event) {
            event.preventDefault();

            const billName = document.getElementById('bill-name').value;
            const billAmount = parseFloat(document.getElementById('bill-amount').value);
            const billDueDate = document.getElementById('bill-due-date').value;

            if (isNaN(billAmount)) {
                alert("Please enter a valid amount.");
                return;
            }

            addBill(billName, billAmount, billDueDate);
            document.getElementById('bill-form').reset();
        });

        function addBill(name, amount, dueDate) {
            const billItem = { name, amount, dueDate };
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.push(billItem);
            bills.sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate)); // Sort by due date
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        function displayBills() {
            const billList = document.getElementById('bill-list');
            billList.innerHTML = '';
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.forEach((bill, index) => {
                const billItem = document.createElement('div');
                billItem.className = 'bill-item';
                billItem.innerHTML = `
                    <span>${bill.name}</span>
                    <span class="amount">£${bill.amount.toFixed(2)}</span>
                    <span>${bill.dueDate}</span>
                    <button onclick="removeBill(${index})">Remove</button>
                `;
                billList.appendChild(billItem);
            });
            calculateTotals();
        }

        function removeBill(index) {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            bills.splice(index, 1);
            localStorage.setItem('bills', JSON.stringify(bills));
            displayBills();
        }

        document.getElementById('income-form').addEventListener('submit', function(event) {
            event.preventDefault();

            const incomeName = document.getElementById('income-name').value;
            const incomeAmount = parseFloat(document.getElementById('income-amount').value);
            const incomeMonth = document.getElementById('income-month').value;

            if (isNaN(incomeAmount)) {
                alert("Please enter a valid amount.");
                return;
            }

            addIncome(incomeName, incomeAmount, incomeMonth);
            document.getElementById('income-form').reset();
        });

        function addIncome(name, amount, month) {
            const incomeItem = { name, amount, month };
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
                    <span>${income.name} - £${income.amount.toFixed(2)} (${income.month})</span>
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

        function calculateTotals() {
            let bills = JSON.parse(localStorage.getItem('bills')) || [];
            let incomes = JSON.parse(localStorage.getItem('incomes')) || [];

            const totalBills = bills.reduce((sum, bill) => sum + bill.amount, 0);
            const currentMonth = new Date().toISOString().slice(0, 7); // Get current month (YYYY-MM)
            const totalIncome = incomes
                .filter(income => income.month === currentMonth)
                .reduce((sum, income) => sum + income.amount, 0);

            document.getElementById('total-amount').textContent = `Total Bills: £${totalBills.toFixed(2)}`;
            document.getElementById('total-income').textContent = `Total Income (This Month): £${totalIncome.toFixed(2)}`;
            document.getElementById('remaining-balance').textContent = `£${(totalIncome - totalBills).toFixed(2)}`;
        }

        displayBills();
        displayIncomes();
    </script>
</body>
</html>