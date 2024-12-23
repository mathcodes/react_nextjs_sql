# Data Visualization App

![Next.js](https://img.shields.io/badge/Framework-Next.js-000000?style=for-the-badge&logo=nextdotjs) ![React](https://img.shields.io/badge/Library-React-61DAFB?style=for-the-badge&logo=react) ![TailwindCSS](https://img.shields.io/badge/CSS-TailwindCSS-38B2AC?style=for-the-badge&logo=tailwindcss) ![SQL Server](https://img.shields.io/badge/Database-SQL%20Server-CC2927?style=for-the-badge&logo=microsoftsqlserver)

## Overview
This project is a **React and Next.js** application designed for **data visualization** using **SQL Server views**. It includes features such as:

- **Dynamic Table Rendering**: Display data from SQL Server views.
- **Data Visualization**: Interactive charts built with **Chart.js**.
- **Export Options**: Export data as **CSV**, with buttons for Excel and TXT compatibility.
- **Dark/Light Mode**: Toggle between themes using **TailwindCSS**.
- **Scalable Architecture**: Suitable for integration into existing ERP and analytics workflows.

---

## Installation
1. Clone the repository:
```bash
git clone https://github.com/yourusername/data-visualization-app.git
cd data-visualization-app
```

2. Install dependencies:
```bash
npm install
```

3. Set up environment variables:
Create a `.env.local` file and add your database credentials:
```
DB_USER=username
DB_PASSWORD=password
DB_SERVER=localhost
DB_DATABASE=your_database
```

4. Start the development server:
```bash
npm run dev
```

---

## Folder Structure
```
my-app/
├── pages/
│   ├── api/
│   │   ├── data.js
│   ├── index.js
├── lib/
│   ├── db.js
├── public/
├── styles/
│   ├── globals.css
├── .env.local
├── package.json
```

---

## Code Examples

### Database Connection (**lib/db.js**):
```javascript
const sql = require('mssql');

const config = {
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  server: process.env.DB_SERVER,
  database: process.env.DB_DATABASE,
  options: {
    encrypt: true, // Use encryption
    trustServerCertificate: true, // Change this for production
  },
};

async function connectToDatabase() {
  try {
    const pool = await sql.connect(config);
    return pool;
  } catch (err) {
    console.error('Database connection failed:', err);
    throw err;
  }
}

module.exports = { connectToDatabase };
```

### API Endpoint (**pages/api/data.js**):
```javascript
import { connectToDatabase } from '../../lib/db';

export default async function handler(req, res) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const db = await connectToDatabase();
    const result = await db.query('SELECT * FROM SalesData'); // Replace with your view
    res.status(200).json(result.recordset);
  } catch (error) {
    console.error('Error fetching data:', error);
    res.status(500).json({ error: 'Failed to fetch data.' });
  }
}
```

### Frontend with Table and Chart (**pages/index.js**):
```javascript
import { useState, useEffect } from 'react';
import axios from 'axios';
import { Line } from 'react-chartjs-2';

export default function Home() {
  const [data, setData] = useState([]);
  const [darkMode, setDarkMode] = useState(false);

  useEffect(() => {
    axios.get('/api/data')
      .then((response) => setData(response.data))
      .catch((error) => console.error(error));
  }, []);

  const toggleDarkMode = () => {
    setDarkMode(!darkMode);
    document.documentElement.classList.toggle('dark');
  };

  const exportToCsv = () => {
    const csv = data.map(row => Object.values(row).join(",")).join("\n");
    downloadFile(csv, 'data.csv');
  };

  const downloadFile = (content, fileName) => {
    const blob = new Blob([content], { type: 'text/csv' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = fileName;
    link.click();
  };

  const chartData = {
    labels: data.map(item => item.date),
    datasets: [
      {
        label: 'Sales ($)',
        data: data.map(item => item.sales),
        borderColor: 'rgb(75, 192, 192)',
        tension: 0.1,
      },
    ],
  };

  return (
    <div className={`min-h-screen ${darkMode ? 'dark bg-gray-900 text-white' : 'bg-white text-black'} p-6`}>
      <button className="mb-4 p-2 bg-blue-500 text-white rounded" onClick={toggleDarkMode}>
        Toggle {darkMode ? 'Light' : 'Dark'} Mode
      </button>

      <h1 className="text-3xl font-bold mb-6">Data Visualization</h1>

      <Line data={chartData} />

      <button onClick={exportToCsv} className="mt-6 p-2 bg-green-500 text-white rounded">
        Export as CSV
      </button>

      <table className="mt-6 table-auto w-full border-collapse border border-gray-300">
        <thead>
          <tr>
            {data.length > 0 && Object.keys(data[0]).map((key) => (
              <th key={key} className="border p-2">{key}</th>
            ))}
          </tr>
        </thead>
        <tbody>
          {data.map((row, idx) => (
            <tr key={idx}>
              {Object.values(row).map((val, i) => (
                <td key={i} className="border p-2">{val}</td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## Contribution Guidelines
- Follow [Conventional Commits](https://www.conventionalcommits.org/) for commit messages.
- Use [Prettier](https://prettier.io/) for code formatting.
- Open issues for bugs or enhancements.

---

## License
This project is licensed under the [MIT License](LICENSE).
