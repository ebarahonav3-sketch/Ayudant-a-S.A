# Ayudant-a-S.A
const sqlite3 = require('sqlite3').verbose();
const path = require('path');
const fs = require('fs');

const dataDir = path.join(__dirname, 'data');
if (!fs.existsSync(dataDir)) fs.mkdirSync(dataDir);

const dbPath = path.join(dataDir, 'db.sqlite');
const db = new sqlite3.Database(dbPath);

function run(sql, params = []) {
  return new Promise((resolve, reject) => {
    db.run(sql, params, function (err) {
      if (err) reject(err);
      else resolve({ id: this.lastID, changes: this.changes });
    });
  });
}
function all(sql, params = []) {
  return new Promise((resolve, reject) => {
    db.all(sql, params, (err, rows) => {
      if (err) reject(err);
      else resolve(rows);
    });
  });
}
function get(sql, params = []) {
  return new Promise((resolve, reject) => {
    db.get(sql, params, (err, row) => {
      if (err) reject(err);
      else resolve(row);
    });
  });
}

// Inicializar tablas
db.serialize(() => {
  db.run(`PRAGMA foreign_keys = ON;`);
  db.run(`CREATE TABLE IF NOT EXISTS customers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT,
    phone TEXT
  );`);
  db.run(`CREATE TABLE IF NOT EXISTS sales (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    customer_id INTEGER,
    date TEXT,
    amount REAL,
    description TEXT,
    FOREIGN KEY(customer_id) REFERENCES customers(id) ON DELETE SET NULL
  );`);
});

module.exports = {
  // Customers
  getAllCustomers: () => all(`SELECT * FROM customers ORDER BY id DESC`),
  getCustomerById: (id) => get(`SELECT * FROM customers WHERE id = ?`, [id]),
  createCustomer: (c) => run(`INSERT INTO customers (name,email,phone) VALUES (?,?,?)`, [c.name, c.email || null, c.phone || null]),
  updateCustomer: (id, c) => run(`UPDATE customers SET name = ?, email = ?, phone = ? WHERE id = ?`, [c.name, c.email || null, c.phone || null, id]),
  deleteCustomer: (id) => run(`DELETE FROM customers WHERE id = ?`, [id]),
  // Sales
  getAllSales: () => all(`SELECT sales.*, customers.name as customer_name FROM sales LEFT JOIN customers ON sales.customer_id = customers.id ORDER BY sales.id DESC`),
  getSaleById: (id) => get(`SELECT * FROM sales WHERE id = ?`, [id]),
  createSale: (s) => run(`INSERT INTO sales (customer_id, date, amount, description) VALUES (?,?,?,?)`, [s.customer_id || null, s.date || new Date().toISOString(), s.amount || 0, s.description || null]),
  updateSale: (id, s) => run(`UPDATE sales SET customer_id = ?, date = ?, amount = ?, description = ? WHERE id = ?`, [s.customer_id || null, s.date || new Date().toISOString(), s.amount || 0, s.description || null, id]),
  deleteSale: (id) => run(`DELETE FROM sales WHERE id = ?`, [id])
};
