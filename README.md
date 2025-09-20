<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Smart Doc Checker</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    :root {
      --primary: #4361ee;
      --secondary: #3a0ca3;
      --accent: #f72585;
      --light: #f8f9fa;
      --dark: #212529;
      --success: #4cc9f0;
      --warning: #ffd166;
      --danger: #ef476f;
      --gray: #adb5bd;
    }
    * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
    body {
      background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
      color: var(--dark); line-height: 1.6; min-height: 100vh; padding: 20px;
    }
    .container { max-width: 1200px; margin: 0 auto; }
    header { text-align: center; margin-bottom: 30px; padding: 20px; background: white;
      border-radius: 12px; box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1); }
    h1 { color: var(--primary); margin-bottom: 10px; font-size: 2.5rem; }
    .subtitle { color: var(--gray); font-size: 1.2rem; max-width: 800px; margin: 0 auto; }
    .dashboard { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 30px; }
    .card { background: white; border-radius: 12px; padding: 25px; box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
    .card h2 { color: var(--secondary); margin-bottom: 20px; padding-bottom: 10px;
      border-bottom: 2px solid var(--light); display: flex; align-items: center; gap: 10px; }
    .upload-area { border: 2px dashed var(--gray); border-radius: 8px; padding: 30px; text-align: center; margin-bottom: 20px; transition: all 0.3s ease; }
    .upload-area:hover { border-color: var(--primary); background-color: #f8f9ff; }
    .upload-area i { font-size: 3rem; color: var(--primary); margin-bottom: 15px; }
    .btn { background: var(--primary); color: white; border: none; padding: 12px 24px;
      border-radius: 8px; cursor: pointer; font-size: 1rem; font-weight: 600;
      transition: all 0.3s ease; display: inline-flex; align-items: center; gap: 8px; }
    .btn:hover { background: var(--secondary); transform: translateY(-2px); }
    .btn-outline { background: transparent; color: var(--primary); border: 2px solid var(--primary); }
    .btn-outline:hover { background: var(--primary); color: white; }
    .file-list { list-style: none; margin-top: 20px; }
    .file-item { display: flex; justify-content: space-between; align-items: center;
      padding: 12px 15px; background: var(--light); border-radius: 8px; margin-bottom: 10px; }
    .file-name { display: flex; align-items: center; gap: 10px; }
    .conflict { padding: 15px; border-left: 4px solid var(--danger); background-color: #fff5f5; border-radius: 4px; margin-bottom: 15px; }
    .conflict h3 { color: var(--danger); margin-bottom: 10px; }
    .suggestion { padding: 15px; border-left: 4px solid var(--success); background-color: #f0f9ff; border-radius: 4px; margin-bottom: 15px; }
    .suggestion h3 { color: var(--success); margin-bottom: 10px; }
    .usage-stats { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-top: 20px; }
    .stat { text-align: center; padding: 15px; background: var(--light); border-radius: 8px; }
    .stat-number { font-size: 2rem; font-weight: 700; color: var(--primary); }
    .stat-label { color: var(--gray); font-size: 0.9rem; }
    .monitoring { display: flex; align-items: center; gap: 15px; padding: 15px; background: #e6f7ff; border-radius: 8px; margin-top: 20px; }
    .status-indicator { width: 12px; height: 12px; border-radius: 50%; background: var(--success); }
    .report { margin-top: 20px; }
    .report-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
    table { width: 100%; border-collapse: collapse; }
    th, td { padding: 12px 15px; text-align: left; border-bottom: 1px solid var(--light); }
    th { background: var(--light); font-weight: 600; }
    tr:hover { background: #f8f9fa; }
    .badge { padding: 5px 10px; border-radius: 20px; font-size: 0.8rem; font-weight: 600; }
    .badge-danger { background: #ffeaea; color: var(--danger); }
    .badge-warning { background: #fff6e0; color: #e6a700; }
    .badge-success { background: #e8f8f2; color: #0cb577; }
    footer { text-align: center; margin-top: 40px; padding: 20px; color: var(--gray); font-size: 0.9rem; }
    @media (max-width: 900px) {
      .dashboard { grid-template-columns: 1fr; }
      .usage-stats { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>
<div class="container">
  <header>
    <h1><i class="fas fa-file-contract"></i> Smart Doc Checker</h1>
    <p class="subtitle">Upload multiple documents, find contradictions, and generate detailed reports automatically</p>
  </header>

  <div class="dashboard">
    <!-- Upload + Stats -->
    <div class="card">
      <h2><i class="fas fa-file-upload"></i> Document Upload</h2>
      <div class="upload-area" id="upload-area">
        <i class="fas fa-cloud-upload-alt"></i>
        <p>Drag & drop your documents here or click to browse</p>
        <input type="file" id="file-upload" multiple style="display: none;" accept=".txt,.pdf,.docx">
        <button class="btn" id="select-files"><i class="fas fa-plus"></i> Select Files</button>
      </div>

      <h3>Uploaded Documents</h3>
      <ul class="file-list" id="file-list"></ul>

      <div class="usage-stats">
        <div class="stat">
          <div class="stat-number" id="doc-count">0</div>
          <div class="stat-label">Documents Uploaded</div>
        </div>
        <div class="stat">
          <div class="stat-number" id="conflict-count">0</div>
          <div class="stat-label">Conflicts Found</div>
        </div>
      </div>
      <button class="btn" id="clear-files" style="margin-top:15px;"><i class="fas fa-trash"></i> Clear All</button>
    </div>

    <!-- Analysis -->
    <div class="card">
      <h2><i class="fas fa-search"></i> Analysis Results</h2>
      <div id="analysis-results">
        <p>No analysis yet. Upload files to begin.</p>
      </div>
      <div class="monitoring">
        <div class="status-indicator" id="status-indicator"></div>
        <div class="monitoring-info">
          <p><strong>Live Monitoring Active</strong> - Tracking changes to external policy documents</p>
        </div>
      </div>
    </div>
  </div>

  <!-- Reports -->
  <div class="card">
    <div class="report-header">
      <h2><i class="fas fa-file-alt"></i> Generated Reports</h2>
      <button class="btn" id="export-json"><i class="fas fa-download"></i> Export JSON</button>
    </div>
    <table>
      <thead>
        <tr>
          <th>Report ID</th>
          <th>Date Generated</th>
          <th>Documents</th>
          <th>Conflicts Found</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody id="report-table"></tbody>
    </table>
  </div>

  <footer>
    <p>Smart Doc Checker • Powered by AI and Pathway Technology</p>
    <p>© 2023 All rights reserved</p>
  </footer>
</div>

<script>
const fileUpload = document.getElementById('file-upload');
const fileList = document.getElementById('file-list');
const selectFiles = document.getElementById('select-files');
const docCount = document.getElementById('doc-count');
const conflictCount = document.getElementById('conflict-count');
const analysisResults = document.getElementById('analysis-results');
const reportTable = document.getElementById('report-table');

let uploadedFiles = [];
let reports = [];
let reportId = 42;

selectFiles.addEventListener('click', () => fileUpload.click());

// File input handler
fileUpload.addEventListener('change', (e) => {
  for (let file of e.target.files) {
    uploadedFiles.push(file);
  }
  renderFileList();
});

// Render file list
function renderFileList() {
  fileList.innerHTML = '';
  uploadedFiles.forEach((file, index) => {
    let li = document.createElement('li');
    li.className = 'file-item';
    li.innerHTML = `
      <div class="file-name"><i class="fas fa-file"></i><span>${file.name}</span></div>
      <div class="file-actions"><button class="btn btn-outline" onclick="analyzeFile(${index})">Analyze</button></div>
    `;
    fileList.appendChild(li);
  });
  docCount.textContent = uploadedFiles.length;
}

// Clear files
document.getElementById('clear-files').addEventListener('click', () => {
  uploadedFiles = [];
  renderFileList();
  analysisResults.innerHTML = '<p>No analysis yet. Upload files to begin.</p>';
  docCount.textContent = "0";
  conflictCount.textContent = "0";
});

// Mock analysis
function analyzeFile(index) {
  const file = uploadedFiles[index];
  const conflicts = Math.floor(Math.random() * 3);
  conflictCount.textContent = parseInt(conflictCount.textContent) + conflicts;

  let resultHTML = `<div class="conflict">
    <h3><i class="fas fa-exclamation-circle"></i> Analyzed ${file.name}</h3>
    <p>Found <strong>${conflicts}</strong> possible conflicts.</p>
  </div>`;
  analysisResults.innerHTML = resultHTML;

  let now = new Date().toLocaleString();
  reports.push({
    id: `#RPT-${reportId++}`,
    date: now,
    docs: 1,
    conflicts: conflicts,
    status: conflicts > 0 ? "Action Required" : "Resolved"
  });
  renderReports();
}

// Reports table
function renderReports() {
  reportTable.innerHTML = '';
  reports.forEach(r => {
    reportTable.innerHTML += `
      <tr>
        <td>${r.id}</td>
        <td>${r.date}</td>
        <td>${r.docs}</td>
        <td>${r.conflicts}</td>
        <td><span class="badge ${r.status==="Resolved"?"badge-success":"badge-danger"}">${r.status}</span></td>
      </tr>`;
  });
}

// Export reports
document.getElementById('export-json').addEventListener('click', () => {
  const blob = new Blob([JSON.stringify(reports, null, 2)], {type: 'application/json'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = "smart_doc_reports.json";
  a.click();
  URL.revokeObjectURL(url);
});

// Monitoring simulation
setInterval(() => {
  const indicator = document.getElementById('status-indicator');
  const monitoringInfo = document.querySelector('.monitoring-info p');

  if (indicator.style.background === 'var(--success)') {
    indicator.style.background = 'var(--warning)';
    monitoringInfo.innerHTML = '<strong>Update Detected</strong> - College rules modified just now';
    alert('External policy update detected! Please re-run analysis.');
  } else {
    indicator.style.background = 'var(--success)';
    monitoringInfo.innerHTML = '<strong>Live Monitoring Active</strong> - Tracking changes to external policy documents';
  }
}, 20000);
</script>
</body>
</html>
