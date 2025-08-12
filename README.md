# byrgh.github.io
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Asset Update Scanner</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 10px; background: #f0f0f0; }
    #video { width: 100%; max-width: 400px; border: 1px solid #ccc; }
    input, button { font-size: 1rem; padding: 8px; margin: 5px 0; width: 100%; max-width: 400px; }
    #items { margin-top: 20px; max-width: 420px; }
    #items table { width: 100%; border-collapse: collapse; }
    #items th, #items td { border: 1px solid #999; padding: 6px; text-align: left; }
  </style>
</head>
<body>

<h2>Asset Update Scanner</h2>

<video id="video" autoplay muted playsinline></video><br/>

<button id="startButton">Start Scanner</button>
<button id="stopButton" disabled>Stop Scanner</button>

<div>
  <label>Service Tag (QR):</label>
  <input id="serviceTag" placeholder="Scan or type..." />
</div>
<div>
  <label>Asset Tag (Barcode):</label>
  <input id="assetTag" placeholder="Scan or type..." />
</div>
<div>
  <label>Model Name:</label>
  <input id="modelName" placeholder="e.g. Latitude 5540" />
</div>
<div>
  <label>Date:</label>
  <input id="date" type="date" />
</div>
<div>
  <label>Technician:</label>
  <input id="technician" placeholder="Your name" />
</div>

<button id="saveButton">Save Item</button>

<div id="items">
  <h3>Saved Items</h3>
  <table>
    <thead><tr><th>Service Tag</th><th>Asset Tag</th><th>Model</th><th>Date</th><th>Technician</th></tr></thead>
    <tbody id="itemsBody"><tr><td colspan="5">No items saved.</td></tr></tbody>
  </table>
</div>

<button id="exportButton">Export as Text</button>

<script src="https://cdn.jsdelivr.net/npm/@zxing/library@0.20.0/umd/index.min.js"></script>
<script>
  const video = document.getElementById('video');
  const startButton = document.getElementById('startButton');
  const stopButton = document.getElementById('stopButton');
  const serviceTagInput = document.getElementById('serviceTag');
  const assetTagInput = document.getElementById('assetTag');
  const modelNameInput = document.getElementById('modelName');
  const dateInput = document.getElementById('date');
  const technicianInput = document.getElementById('technician');
  const saveButton = document.getElementById('saveButton');
  const itemsBody = document.getElementById('itemsBody');
  const exportButton = document.getElementById('exportButton');

  let selectedDeviceId;
  let codeReader = null;
  let savedItems = [];

  // Set today's date in the date input
  function setToday() {
    const d = new Date();
    const iso = d.toISOString().split('T')[0];
    dateInput.value = iso;
  }
  setToday();

  // Render saved items table
  function renderItems() {
    if (savedItems.length === 0) {
      itemsBody.innerHTML = '<tr><td colspan="5">No items saved.</td></tr>';
      return;
    }
    itemsBody.innerHTML = savedItems.map(item => `
      <tr>
        <td>${item.serviceTag || ''}</td>
        <td>${item.assetTag || ''}</td>
        <td>${item.modelName || ''}</td>
        <td>${item.date || ''}</td>
        <td>${item.technician || ''}</td>
      </tr>`).join('');
  }

  // Save current form values into savedItems array
  saveButton.onclick = () => {
    const newItem = {
      serviceTag: serviceTagInput.value.trim(),
      assetTag: assetTagInput.value.trim(),
      modelName: modelNameInput.value.trim(),
      date: dateInput.value,
      technician: technicianInput.value.trim()
    };
    if (!newItem.serviceTag && !newItem.assetTag) {
      alert('Please enter or scan a Service Tag or Asset Tag.');
      return;
    }
    savedItems.push(newItem);
    renderItems();
    serviceTagInput.value = '';
    assetTagInput.value = '';
    modelNameInput.value = '';
    setToday();
  };

  // Export saved items as plain text for easy copy/paste
  exportButton.onclick = () => {
    if (savedItems.length === 0) {
      alert('No items to export.');
      return;
    }
    const text = savedItems.map(item =>
      `Service Tag: ${item.serviceTag}, Asset Tag: ${item.assetTag}, Model: ${item.modelName}, Date: ${item.date}, Technician: ${item.technician}`
    ).join('\n');
    // Open text in a new tab for easy copy
    const win = window.open('', '_blank');
    win.document.write('<pre>' + text + '</pre>');
    win.document.close();
  };

  // Start the camera and scanning
  startButton.onclick = async () => {
    codeReader = new ZXing.BrowserMultiFormatReader();
    try {
      const devices = await codeReader.listVideoInputDevices();
      selectedDeviceId = devices[devices.length - 1].deviceId; // use back camera
      await codeReader.decodeFromVideoDevice(selectedDeviceId, video, (result, err) => {
        if (result) {
          const text = result.getText();
          const format = result.getBarcodeFormat();
          // Heuristic: QR code = Service Tag, others = Asset Tag
          if (format === 'QR_CODE') {
            serviceTagInput.value = text;
          } else {
            assetTagInput.value = text;
          }
          codeReader.reset();
          stopButton.disabled = true;
          startButton.disabled = false;
          video.srcObject && video.srcObject.getTracks().forEach(t => t.stop());
        }
      });
      startButton.disabled = true;
      stopButton.disabled = false;
    } catch (e) {
      alert('Error starting camera: ' + e);
    }
  };

  // Stop the camera and scanning
  stopButton.onclick = () => {
    if (codeReader) {
      codeReader.reset();
      codeReader = null;
    }
    video.srcObject && video.srcObject.getTracks().forEach(t => t.stop());
    startButton.disabled = false;
    stopButton.disabled = true;
  };

  renderItems();
</script>

</body>
</html>

