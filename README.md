 body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      margin: 0;
      padding: 0;
      background-color: #f3f8fb; /* خلفية فاتحة */
      color: #333;
    }

    h1 {
      background-color: #4a90e2; /* أزرق أنيق */
      color: white;
      padding: 20px;
      margin: 0;
      text-align: center;
      font-size: 1.8em;
    }

    .container {
      display: flex;
      justify-content: space-around;
      align-items: flex-start;
      flex-wrap: wrap;
      gap: 20px;
      padding: 20px;
    }

    .section {
      background: #fff; /* خلفية بيضاء */
      border-radius: 10px;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
      padding: 20px;
      flex: 1 1 calc(50% - 20px);
      max-width: 500px;
      text-align: center;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin: 20px 0;
    }

    th, td {
      padding: 12px;
      text-align: center;
      border: 1px solid #ddd;
    }

    th {
      background-color: #4a90e2;
      color: white;
    }

    td {
      background-color: #f9f9f9;
    }

    .update-form {
      display: flex;
      justify-content: center;
      gap: 10px;
      margin-top: 10px;
    }

    .update-form input[type="number"] {
      padding: 5px;
      font-size: 1em;
      width: 80px;
      border: 1px solid #ccc;
      border-radius: 5px;
    }

    .update-form button {
      padding: 5px 10px;
      background-color: #4a90e2;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s;
    }

    .update-form button:hover {
      background-color: #357ab7;
    }

    canvas {
      max-width: 100%;
      margin: 0 auto;
      margin-top: 20px;
    }

    @media (max-width: 768px) {
      .section {
        flex: 1 1 100%;
      }
    }
