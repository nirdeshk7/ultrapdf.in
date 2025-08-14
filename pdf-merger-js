// server.js

const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const { exec } = require('child_process');
const PDFMerger = require('pdf-merger-js');

const app = express();
const upload = multer({ dest: 'uploads/' });

// Serve frontend static files from "public" folder
app.use(express.static('public'));

// POST /merge: receive files, convert non-PDF to PDF, merge all PDFs, send merged file
app.post('/merge', upload.array('files', 500), async (req, res) => {
  try {
    const uploadedFiles = req.files;
    const convertedFiles = [];

    // Convert non-PDF files to PDF using LibreOffice CLI
    async function convertToPDF(filePath) {
      return new Promise((resolve, reject) => {
        const outDir = path.dirname(filePath);
        exec(`libreoffice --headless --convert-to pdf --outdir ${outDir} ${filePath}`, (error) => {
          if (error) {
            return reject(error);
          }
          const pdfPath = filePath.replace(path.extname(filePath), '.pdf');
          resolve(pdfPath);
        });
      });
    }

    // Convert files if needed, else keep PDFs as is
    for (const file of uploadedFiles) {
      if (path.extname(file.originalname).toLowerCase() === '.pdf') {
        convertedFiles.push(file.path);
      } else {
        const pdfPath = await convertToPDF(file.path);
        convertedFiles.push(pdfPath);
      }
    }

    // Merge all PDFs
    const merger = new PDFMerger();
    for (const pdfPath of convertedFiles) {
      merger.add(pdfPath);
    }

    const outputPath = `merged_${Date.now()}.pdf`;
    await merger.save(outputPath);

    // Send merged PDF as download
    res.download(outputPath, 'merged.pdf', (err) => {
      // Cleanup: delete uploaded and merged files
      uploadedFiles.forEach(f => fs.unlinkSync(f.path));
      convertedFiles.forEach(f => fs.existsSync(f) && fs.unlinkSync(f));
      if (fs.existsSync(outputPath)) fs.unlinkSync(outputPath);
      if (err) console.error('Download error:', err);
    });

  } catch (error) {
    console.error('Error during merging:', error);
    res.status(500).json({ error: 'Failed to merge files' });
  }
});

// Start server on PORT 3000 or environment port
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`ultaPDF server running on port ${PORT}`);
});
