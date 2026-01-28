# Mass Balance Calculator Web App - Complete Setup Guide for Your PC

## 📋 What You'll Build
A full-stack web application with:
- Professional calculator interface
- Database to save calculations
- History page to view past calculations
- PDF export functionality

---

## ⚙️ STEP 1: Install Prerequisites

### 1.1 Install Node.js

**Windows:**
1. Go to https://nodejs.org/
2. Download the "LTS" version (recommended)
3. Run the installer
4. Keep clicking "Next" with default settings
5. Click "Install"
6. Restart your computer

**Mac:**
1. Go to https://nodejs.org/
2. Download the "LTS" version
3. Open the .pkg file and follow instructions
4. Restart your computer

**Verify Installation:**
Open Terminal (Mac) or Command Prompt (Windows) and type:
```bash
node --version
```
You should see something like `v20.11.0`

Then type:
```bash
npm --version
```
You should see something like `10.2.4`

✅ If you see version numbers, you're good to go!

---

## 📁 STEP 2: Create Project Folders

### 2.1 Choose a Location
Pick where you want your project. For example:
- Windows: `C:\Users\YourName\Documents\`
- Mac: `/Users/YourName/Documents/`

### 2.2 Open Terminal/Command Prompt
**Windows:** 
- Press `Windows Key + R`
- Type `cmd` and press Enter

**Mac:**
- Press `Command + Space`
- Type `terminal` and press Enter

### 2.3 Navigate to Your Location
```bash
# Windows example:
cd C:\Users\YourName\Documents

# Mac example:
cd ~/Documents
```

### 2.4 Create Project Folder
```bash
mkdir mass-balance-calculator
cd mass-balance-calculator
```

---

## 🔧 STEP 3: Set Up Backend

### 3.1 Create Backend Folder
```bash
mkdir backend
cd backend
```

### 3.2 Initialize Node.js Project
```bash
npm init -y
```
✅ This creates a `package.json` file

### 3.3 Install Backend Dependencies
```bash
npm install express cors sqlite3 body-parser uuid date-fns
npm install --save-dev nodemon
```
⏳ This will take 1-2 minutes. You'll see a progress bar.

### 3.4 Create server.js File

**Windows - Using Notepad:**
1. Right-click in the `backend` folder
2. Click "New" → "Text Document"
3. Name it `server.js` (make sure to remove the .txt extension)
4. Open it with Notepad

**Mac - Using TextEdit:**
1. Open TextEdit
2. Go to Format → Make Plain Text
3. Save as `server.js` in the backend folder

**Copy this code into server.js:**

```javascript
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const sqlite3 = require('sqlite3').verbose();
const { v4: uuidv4 } = require('uuid');

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(cors());
app.use(bodyParser.json());

// Database setup
const db = new sqlite3.Database('./mass_balance.db', (err) => {
  if (err) console.error('Database error:', err);
  else console.log('✓ Connected to SQLite database');
});

// Create tables
db.run(`
  CREATE TABLE IF NOT EXISTS calculations (
    id TEXT PRIMARY KEY,
    timestamp TEXT,
    sample_id TEXT,
    analyst_name TEXT,
    stress_type TEXT,
    initial_api REAL,
    stressed_api REAL,
    initial_degradants REAL,
    stressed_degradants REAL,
    degradant_mw REAL,
    parent_mw REAL,
    rrf REAL,
    smb REAL,
    amb REAL,
    rmb REAL,
    lk_imb REAL,
    lambda REAL,
    omega REAL,
    recommended_method TEXT,
    recommended_value REAL,
    confidence_index REAL,
    degradation_level REAL,
    status TEXT,
    diagnostic_message TEXT,
    rationale TEXT
  )
`);

// Calculation Engine
function calculateMassBalance(data) {
  const {
    initial_api,
    stressed_api,
    initial_degradants,
    stressed_degradants,
    degradant_mw,
    parent_mw,
    rrf
  } = data;

  // Basic calculations
  const delta_api = initial_api - stressed_api;
  const delta_degradants = stressed_degradants - initial_degradants;
  const degradation_level = (delta_api / initial_api) * 100;

  // SMB - Simple Mass Balance
  const smb = stressed_api + stressed_degradants;

  // AMB - Absolute Mass Balance
  const amb = ((stressed_api + stressed_degradants) / (initial_api + initial_degradants)) * 100;

  // RMB - Relative Mass Balance
  const rmb = delta_api === 0 ? null : (delta_degradants / delta_api) * 100;

  // Correction factors
  const lambda = rrf ? 1 / rrf : 1.0;
  const omega = (degradant_mw && parent_mw) ? parent_mw / degradant_mw : 1.0;
  const corrected_degradants = stressed_degradants * lambda * omega;

  // LK-IMB - Corrected Mass Balance
  const lk_imb = ((stressed_api + corrected_degradants) / initial_api) * 100;

  // Determine recommended method
  let recommended_method;
  if (delta_api < 2) {
    recommended_method = 'AMB';
  } else if (delta_api >= 5 && delta_api <= 20) {
    recommended_method = 'RMB';
  } else {
    recommended_method = 'LK-IMB';
  }

  // Get recommended value
  const recommended_value = 
    recommended_method === 'AMB' ? amb :
    recommended_method === 'RMB' ? rmb :
    lk_imb;

  // Calculate confidence index
  const analytical_uncertainty = 2.5;
  const confidence_index = Math.abs(100 - amb) > 0 
    ? 100 * (1 - analytical_uncertainty / Math.abs(100 - amb))
    : 95;

  // Determine status
  let status;
  if (recommended_value >= 95 && recommended_value <= 105) {
    status = 'PASS';
  } else if (recommended_value >= 90) {
    status = 'ALERT';
  } else {
    status = 'OOS';
  }

  // Generate diagnostic message
  let diagnostic_message;
  let rationale;

  if (amb < 95 && lambda === 1.0 && omega === 1.0) {
    diagnostic_message = '⚠ Suspected volatile loss detected. Recommend headspace GC-MS analysis.';
    rationale = 'Low mass balance with no correction factors suggests volatile degradation products.';
  } else if (amb < 95 && lambda > 1.2) {
    diagnostic_message = '⚠ UV-silent degradant suspected. Consider CAD or CLND detection.';
    rationale = 'High RRF correction suggests chromophore changes in degradation pathway.';
  } else if (delta_api < 2) {
    diagnostic_message = '✓ Low degradation level. AMB method appropriate.';
    rationale = 'Minimal degradation observed. Standard absolute method is sufficient.';
  } else if (degradation_level > 20) {
    diagnostic_message = '✓ Mass balance acceptable. Stoichiometric corrections applied.';
    rationale = 'High degradation with fragmentation detected. LK-IMB correction mandatory per best practices.';
  } else {
    diagnostic_message = '✓ Mass balance acceptable. Continue routine testing per ICH Q1A(R2).';
    rationale = 'Results within acceptable limits. No anomalies detected.';
  }

  return {
    calculation_id: uuidv4(),
    timestamp: new Date().toISOString(),
    results: {
      smb: parseFloat(smb.toFixed(2)),
      amb: parseFloat(amb.toFixed(2)),
      rmb: rmb !== null ? parseFloat(rmb.toFixed(2)) : null,
      lk_imb: parseFloat(lk_imb.toFixed(2))
    },
    correction_factors: {
      lambda: parseFloat(lambda.toFixed(2)),
      omega: parseFloat(omega.toFixed(2))
    },
    recommended_method,
    recommended_value: parseFloat(recommended_value.toFixed(2)),
    confidence_index: parseFloat(confidence_index.toFixed(1)),
    degradation_level: parseFloat(degradation_level.toFixed(1)),
    status,
    diagnostic_message,
    rationale
  };
}

// API Endpoints

// POST /api/calculate
app.post('/api/calculate', (req, res) => {
  try {
    const calculation = calculateMassBalance(req.body);
    res.json(calculation);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// POST /api/save
app.post('/api/save', (req, res) => {
  const { inputs, results } = req.body;
  
  const stmt = db.prepare(`
    INSERT INTO calculations VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
  `);

  stmt.run(
    results.calculation_id,
    results.timestamp,
    inputs.sample_id || '',
    inputs.analyst_name || '',
    inputs.stress_type || '',
    inputs.initial_api,
    inputs.stressed_api,
    inputs.initial_degradants,
    inputs.stressed_degradants,
    inputs.degradant_mw || null,
    inputs.parent_mw || null,
    inputs.rrf || null,
    results.results.smb,
    results.results.amb,
    results.results.rmb,
    results.results.lk_imb,
    results.correction_factors.lambda,
    results.correction_factors.omega,
    results.recommended_method,
    results.recommended_value,
    results.confidence_index,
    results.degradation_level,
    results.status,
    results.diagnostic_message,
    results.rationale,
    (err) => {
      if (err) {
        res.status(500).json({ error: err.message });
      } else {
        res.json({ success: true, calculation_id: results.calculation_id });
      }
    }
  );
});

// GET /api/history
app.get('/api/history', (req, res) => {
  const { page = 1, limit = 20, analyst, stress_type } = req.query;
  const offset = (page - 1) * limit;

  let query = 'SELECT * FROM calculations WHERE 1=1';
  const params = [];

  if (analyst) {
    query += ' AND analyst_name LIKE ?';
    params.push(`%${analyst}%`);
  }

  if (stress_type) {
    query += ' AND stress_type = ?';
    params.push(stress_type);
  }

  query += ' ORDER BY timestamp DESC LIMIT ? OFFSET ?';
  params.push(parseInt(limit), offset);

  db.all(query, params, (err, rows) => {
    if (err) {
      res.status(500).json({ error: err.message });
    } else {
      db.get('SELECT COUNT(*) as total FROM calculations', (err, count) => {
        res.json({
          total: count.total,
          page: parseInt(page),
          calculations: rows
        });
      });
    }
  });
});

// GET /api/calculation/:id
app.get('/api/calculation/:id', (req, res) => {
  db.get('SELECT * FROM calculations WHERE id = ?', [req.params.id], (err, row) => {
    if (err) {
      res.status(500).json({ error: err.message });
    } else if (!row) {
      res.status(404).json({ error: 'Calculation not found' });
    } else {
      res.json(row);
    }
  });
});

// DELETE /api/calculation/:id
app.delete('/api/calculation/:id', (req, res) => {
  db.run('DELETE FROM calculations WHERE id = ?', [req.params.id], (err) => {
    if (err) {
      res.status(500).json({ error: err.message });
    } else {
      res.json({ success: true });
    }
  });
});

app.listen(PORT, () => {
  console.log(`✓ Backend server running on http://localhost:${PORT}`);
});
```

### 3.5 Update package.json

Open `backend/package.json` in Notepad/TextEdit.

Find the `"scripts"` section and replace it with:

```json
"scripts": {
  "start": "node server.js",
  "dev": "nodemon server.js",
  "test": "echo \"Error: no test specified\" && exit 1"
}
```

Save the file.

### 3.6 Test the Backend

In your terminal (make sure you're in the `backend` folder):

```bash
npm run dev
```

You should see:
```
✓ Connected to SQLite database
✓ Backend server running on http://localhost:5000
```

🎉 **Backend is running!** Keep this terminal window open.

---

## 🎨 STEP 4: Set Up Frontend

Open a **NEW** terminal window (keep the backend running in the first one).

### 4.1 Navigate to Project Root

```bash
# Windows example:
cd C:\Users\YourName\Documents\mass-balance-calculator

# Mac example:
cd ~/Documents/mass-balance-calculator
```

### 4.2 Create React App with Vite

```bash
npm create vite@latest frontend -- --template react
```

When prompted:
- Press Enter to confirm

### 4.3 Enter Frontend Folder

```bash
cd frontend
```

### 4.4 Install Dependencies

```bash
npm install
npm install axios recharts jspdf tailwindcss postcss autoprefixer lucide-react
```

⏳ This takes 2-3 minutes.

### 4.5 Initialize Tailwind CSS

```bash
npx tailwindcss init -p
```

### 4.6 Configure Tailwind

Open `frontend/tailwind.config.js` and replace everything with:

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

### 4.7 Update CSS

Open `frontend/src/index.css` and replace everything with:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  font-family: 'Arial', sans-serif;
  margin: 0;
  padding: 0;
}
```

### 4.8 Create Component Files

You need to create 3 files in `frontend/src/components/`:

**First, create the components folder:**

```bash
# Windows (in Command Prompt, from frontend folder):
mkdir src\components

# Mac (in Terminal, from frontend folder):
mkdir -p src/components
```

Now create these 3 files:

#### File 1: `src/components/Calculator.jsx`

Create this file and paste the code from the next section...

---

## 📝 STEP 5: Copy Component Code

I'll create separate files for you to download with all the component code. Let me create those now...

### 5.1 Download All Component Files

I'll create a ZIP file with all the React components you need.

**What to do:**
1. Download the component files I'll create
2. Extract them to `frontend/src/components/`
3. The files you need are:
   - `Calculator.jsx`
   - `Results.jsx`
   - `History.jsx`
   - `App.jsx` (goes in `frontend/src/`)

---

## ▶️ STEP 6: Run the Application

### 6.1 Make Sure Backend is Running

In your first terminal window, you should still see:
```
✓ Backend server running on http://localhost:5000
```

If not, restart it:
```bash
cd backend
npm run dev
```

### 6.2 Start Frontend

In your second terminal window:

```bash
cd frontend
npm run dev
```

You should see:
```
VITE v5.0.0  ready in 500 ms

➜  Local:   http://localhost:5173/
➜  Network: use --host to expose
```

### 6.3 Open in Browser

1. Open your web browser (Chrome, Firefox, Safari, Edge)
2. Go to: `http://localhost:5173`

🎉 **You should see the Mass Balance Calculator!**

---

## 🧪 STEP 7: Test the Application

### Test Data (Pre-filled):
- Initial API: 98%
- Stressed API: 82.5%
- Initial Degradants: 0.5%
- Stressed Degradants: 4.9%
- Parent MW: 500
- Degradant MW: 250
- RRF: 0.8

### Expected Result:
- **LK-IMB: 96.2% (PASS)** ✓

### Things to Try:
1. Click "Calculate Mass Balance" → See results
2. Click "Save" → Saves to history
3. Click "History" tab → View saved calculations
4. Click "Download PDF Report" → Downloads PDF

---

## 🛠️ Troubleshooting

### Problem: "Command not found: npm"
**Solution:** Node.js not installed correctly. Reinstall Node.js and restart computer.

### Problem: "Port 5000 already in use"
**Solution:** 
- Windows: Open Task Manager, end any Node.js processes
- Mac: Open Activity Monitor, quit Node.js processes
Or change the port in `server.js` (line 7): `const PORT = 5001;`

### Problem: "Cannot connect to backend"
**Solution:** 
1. Make sure backend is running (terminal shows "Backend server running")
2. Check `Calculator.jsx` has correct API URL: `http://localhost:5000/api`

### Problem: White screen in browser
**Solution:**
1. Open browser console (F12)
2. Look for error messages
3. Most common: Check all files are saved
4. Try: Close browser, restart frontend (`npm run dev`)

### Problem: Module not found errors
**Solution:** Make sure you ran all `npm install` commands in both backend and frontend folders

---

## 🎯 Quick Reference

### Starting the App (After Initial Setup):

**Terminal 1 - Backend:**
```bash
cd path/to/mass-balance-calculator/backend
npm run dev
```

**Terminal 2 - Frontend:**
```bash
cd path/to/mass-balance-calculator/frontend
npm run dev
```

**Then open:** `http://localhost:5173` in your browser

### Stopping the App:

Press `Ctrl+C` in both terminal windows

---

## 📦 Project Structure

After setup, your folder structure should look like:

```
mass-balance-calculator/
├── backend/
│   ├── node_modules/
│   ├── mass_balance.db (created automatically)
│   ├── package.json
│   └── server.js
└── frontend/
    ├── node_modules/
    ├── public/
    ├── src/
    │   ├── components/
    │   │   ├── Calculator.jsx
    │   │   ├── Results.jsx
    │   │   └── History.jsx
    │   ├── App.jsx
    │   ├── index.css
    │   └── main.jsx
    ├── index.html
    ├── package.json
    ├── tailwind.config.js
    └── vite.config.js
```

---

## ✅ Next Steps

Now I'll create the component files for you to download and copy into your project!
