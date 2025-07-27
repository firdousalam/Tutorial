# step 1 : 
install all dependency

npm install express jspdf fd path url jspdf-autotable nodemon

# step 2 
create one pdfGenerate.js file and run express server

import express from 'express';

// Create an instance of an Express application
const app = express();

// Define the port the server will listen on
const port = 3000;

// Set up a route for the root URL ('/')
app.get('/', (req, res) => {
res.send('Hello World!'); // Send a response to the client
});

// Start the server and listen on the specified port
app.listen(port, () => {
console.log(`Server is running on http://localhost:${port}`);
});

# step 3 
add all dependency and code in pdfGenerate.js

import { jsPDF } from "jspdf";
import fs from 'fs';
import path from 'path';
import { fileURLToPath } from 'url';
import autoTable from 'jspdf-autotable';




const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// Bill Data
const billData = {
  customer: {
    name: 'John Doe',
    address: '456 Green Street\nMumbai, India'
  },
  company: {
    name: 'GetPhysio.in',
    address: '123 Main Road, Kolkata, India\nPhone: +91-9876543210\nEmail: contact@GetPhysio.in'
  },
  date: new Date().toLocaleDateString(),
  billNo: 'INV-2025-0001',
  discount: 100, // Flat discount
  paid: 500,     // Paid amount
  doctor : {
    name: 'Dr. Smith',
    specialization: 'Physiotherapy',
    degree: 'MPT',
    contact : '+91-9876543210',
    registrationNo : 'PT-123456'
  },
  items: [
    { item: 'Consultation', qty: 1, price: 500 },
    { item: 'Physiotherapy', qty: 2, price: 300 },
      { item: 'Consultation', qty: 1, price: 500 },
    { item: 'Physiotherapy', qty: 2, price: 300 },
      { item: 'Consultation', qty: 1, price: 500 },
    { item: 'Physiotherapy', qty: 2, price: 300 },
      { item: 'Consultation', qty: 1, price: 500 },
    { item: 'Physiotherapy', qty: 2, price: 300 }
  ]
};

# step 4 add function to generate PDF

function generateBillPDF(filePath,billData) {
  const doc = new jsPDF();

  const grandTotal = billData.items.reduce((sum, i) => sum + i.qty * i.price, 0);
  const discountAmount = billData.discount || 0;
  const netAmount = grandTotal - discountAmount;
  const paidAmount = billData.paid || 0;
  const balanceAmount = netAmount - paidAmount;
  const currency = amount => `RS.${amount.toFixed(2)}`;
  // Prepare item table data
 const itemTable = billData.items.map(i => [
                                              i.item,
                                              i.qty,
                                              `Rs.${i.price.toFixed(2)}`,
                                              `Rs.${(i.qty * i.price).toFixed(2)}`
                                            ]);

  // ✅ Title

  doc.setFontSize(12).setFont('helvetica', 'bold').text(`${billData.company.name}`, 105, 20, { align: 'center' });
    doc.setFontSize(10).setFont('helvetica', 'bold').text(`${billData.company.address}`, 105, 28, { align: 'center' });
  // ✅ Company & Customer Info Table
  autoTable(doc, {
    startY: 40,
    head: [['Doctor', 'To (Customer)', 'Date','Bill No']],
    body: [
      [
        `${billData.doctor.name} , ${billData.doctor.specialization} ${billData.doctor.degree}\nContact: ${billData.doctor.contact}\nRegistration No: ${billData.doctor.registrationNo}`,
        `${billData.customer.name}\n${billData.customer.address}`,
        billData.date,
        billData.billNo
      ]
    ],
    theme: 'grid',
    styles: { fontSize: 10, cellPadding: 3 },
    headStyles: { fillColor: [220, 220, 220], textColor: 20, fontStyle: 'bold' },
  });

 

  const formattedGrandTotal = `₹${grandTotal.toFixed(2)}`;

  autoTable(doc, {
    startY: doc.lastAutoTable.finalY + 10,
    head: [['Item', 'Quantity', 'Price', 'Total']],
    body: itemTable,
    theme: 'grid',
    styles: { fontSize: 11, cellPadding: 4 },
    headStyles: { fillColor: [200, 200, 200], textColor: 0, fontStyle: 'bold' },
    columnStyles: {
      2: { halign: 'right' },
      3: { halign: 'right' }
    }
  });

// Get the dynamic Y position after the item table
let finalY = doc.lastAutoTable.finalY || 100;
if(finalY<200) {
 // doc.line(10, finalY + 5, 200, finalY + 5); // Draw a line below the table
  finalY = 230;
}
let y = finalY + 20; // Add some padding


  autoTable(doc, {
    startY: y-20,
    head: [['Summary']],
    body: [],
    theme: 'grid',
    styles: { fontSize: 11, cellPadding: 4 },
    headStyles: { fillColor: [200, 200, 200], textColor: 0, fontStyle: 'bold' },
    columnStyles: {
      2: { halign: 'right' },
      3: { halign: 'right' }
    }
  });


 //doc.line(10, finalY + 5, 200, finalY + 5);
    doc.setFontSize(10).setFont('helvetica', 'bold');
    doc.text(`Subtotal : ${currency(grandTotal)}`, 193, y, { align: 'right' });
    y += 8;
    doc.setFontSize(10).setFont('helvetica', 'bold');
    doc.text(`Discount : ${currency(discountAmount)}`, 191, y, { align: 'right' });
    doc.setFontSize(10).setFont('helvetica', 'bold');
    y += 8;
    doc.text(`Net Amount : ${currency(netAmount)}`, 192, y, { align: 'right' });
    doc.setFontSize(10).setFont('helvetica', 'bold');
    y += 8;
    doc.text(`Paid Amount : ${currency(paidAmount)}`, 190, y, { align: 'right' });
    doc.setFontSize(10).setFont('helvetica', 'bold');
    y += 8;
    doc.text(`Balance Due : ${currency(balanceAmount)}`, 190, y, { align: 'right' });
    doc.setFontSize(10).setFont('helvetica', 'bold');
 //doc.line(10, y + 5, 200, y + 5);

 // After all content
  //addWatermark(doc, 'PAID BY CUSTOMER');
  // ✅ Save PDF
  const pdfData = doc.output();
  fs.writeFileSync(filePath, pdfData, 'binary');
}

# step 5 add watermark to PDF 

// function to add watermark
 function addWatermark(doc, text = 'PAID') {
    const pageWidth = doc.internal.pageSize.getWidth();
    const pageHeight = doc.internal.pageSize.getHeight();

    doc.setFontSize(50);
    doc.setTextColor(150, 150, 150); // light gray
    doc.setFont('helvetica', 'bold');
    doc.setTextColor(200); // Light gray
    doc.setTextColor(150, 150, 150);
    doc.setDrawColor(255, 255, 255);

    // Rotate and place the watermark in the center
    doc.saveGraphicsState();
    doc.setGState(new doc.GState({ opacity: 0.15 })); // transparent
    doc.text(text, pageWidth / 2, pageHeight / 1.5, {
      align: 'center',
      angle: 30,
    });
    doc.restoreGraphicsState();
  }
 
# step 6 create route to Create and Download PDF

app.get('/download-pdf', (req, res) => {
  const filePath = path.join(__dirname, 'bill_receipt.pdf');
  generateBillPDF(filePath,billData);

  res.download(filePath, 'bill_receipt.pdf', (err) => {
    if (err) {
      console.error('Error sending file:', err);
      res.status(500).send('Error sending PDF');
    } else {
      // Optionally delete the file after sending
      fs.unlink(filePath, () => {});
    }
  });
});

# step 7
go to package.json
 "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node pdfGenerate.js",
    "dev": "nodemon pdfGenerate.js"
  },


