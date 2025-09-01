# Cxzzz
// Assumptions: These variables should be defined within your code's scope
// Replace the example values/comments with your actual data and HTML content.

let subtotal = 0; // Initialize subtotal
const customerName = "Your Customer Name Here"; // Example: "John Doe"
const totalShippingCost = 50.00; // Example: This should be dynamically calculated or provided
const headerHtml = `
    <!-- Your HTML header content here, e.g., company logo, address -->
    <div style="text-align: center; margin-bottom: 30px;">
        <img src="your_company_logo.png" alt="Company Logo" style="max-width: 150px; height: auto;">
        <h1 style="color: #4472C4; font-family: Arial, sans-serif;">Quotation</h1>
        <p style="font-family: Arial, sans-serif; font-size: 14px;">Your Company Name | Your Address | Your Phone | Your Email</p>
    </div>
`;
const customerHtml = `
    <!-- Your HTML customer information here -->
    <div style="font-family: Arial, sans-serif; margin-bottom: 30px; border: 1px solid #eee; padding: 15px;">
        <h2 style="color: #4472C4; margin-top: 0; font-size: 1.3em;">Customer Details:</h2>
        <p><strong>Client Name:</strong> ${customerName}</p>
        <p><strong>Contact:</strong> 123-456-7890</p>
        <p><strong>Email:</strong> client@example.com</p>
        <p><strong>Address:</strong> 123 Main St, Anytown, Country</p>
        <p><strong>Quotation Date:</strong> ${new Date().toLocaleDateString('en-US')}</p>
    </div>
`;

// Function to format numbers, assumed to be defined
function formatNumber(num) {
    return num.toFixed(2); // Format number to two decimal places
}

// Example data for 'someArray'. Replace this with the actual array of your items.
const someArray = [
    { name: 'wpc-aluminum frame non-L', h: 200, w: 90, qty: 1, unitPrice: 0, totalPrice: 0, data: {}, selectedAddons: {}, itemNotes: 'Front Door' },
    { name: 'wpc glass', h: 210, w: 95, qty: 2, unitPrice: 0, totalPrice: 0, data: {}, selectedAddons: {}, itemNotes: 'Bathroom Door' },
    { name: 'wpc standard door', h: 230, w: 110, qty: 1, unitPrice: 100, totalPrice: 100, data: {}, selectedAddons: { curtain: true }, itemNotes: 'Main Bedroom' },
    { name: 'wpc small door', h: 200, w: 120, qty: 1, unitPrice: 80, totalPrice: 80, data: {}, selectedAddons: { netType: 'Mosquito' }, itemNotes: 'Kids Room' },
    { name: 'wpc slide door', h: 200, w: 150, qty: 1, unitPrice: 120, totalPrice: 120, data: {}, selectedAddons: {}, itemNotes: 'Balcony Slide' }, // This one will not be affected by the area increment rules
    { name: 'another item', h: 180, w: 80, qty: 1, unitPrice: 50, totalPrice: 50, data: {}, selectedAddons: {}, itemNotes: 'Storage Door' }
];

let tableHtml = `<table width="100%" style="border-collapse: collapse; font-family: Arial, sans-serif; font-size: 14px; margin-bottom: 25px;">
                <thead>
                  <tr style="background-color: #4472C4; color: white;">
                    <th style="padding: 10px; text-align: left;">Code/Ref</th>
                    <th style="padding: 10px;">H (cm)</th>
                    <th style="padding: 10px;">W (cm)</th>
                    <th style="padding: 10px;">Area (cm²)</th>
                    <th style="padding: 10px;">Qty</th>
                    <th style="padding: 10px;"></th>
                    <th style="padding: 10px;"></th>
                    <th style="padding: 10px;">Unit Price (OMR)</th>
                    <th style="padding: 10px;">Total Price (OMR)</th>
                    <th style="padding: 10px; text-align: left;">Description</th>
                  </tr>
                </thead>
                <tbody>`;

someArray.forEach((r, index) => {
    // --- Start of required modifications (fixed price adjustments first) ---

    // 1. Adjust price of "wpc-aluminum frame non-L" to 68 OMR
    if (r.name === 'wpc-aluminum frame non-L') {
        r.unitPrice = 68;
        r.totalPrice = r.unitPrice * r.qty;
    }

    // 2. Adjust price of "wpc glass" to 55 OMR
    if (r.name === 'wpc glass') {
        r.unitPrice = 55;
        r.totalPrice = r.unitPrice * r.qty;
    }

    // --- Start of additional area calculation for WPC doors (excluding wpc slide) ---
    const isWpcDoor = r.name.toLowerCase().includes('wpc') && !r.name.toLowerCase().includes('slide');

    if (isWpcDoor) {
        // Check the more specific condition first: if both height (2.2m = 220cm) and width (1m = 100cm) are exceeded
        if (r.h > 220 && r.w > 100) {
            const excessHInCm = r.h - 220; // Excess height in centimeters
            const excessWInCm = r.w - 100; // Excess width in centimeters

            // Convert excess to meters for area calculation in square meters
            const excessHInMeters = excessHInCm / 100;
            const excessWInMeters = excessWInCm / 100;

            const additionalCostFromArea = excessHInMeters * excessWInMeters * 3; // Excess area (m²) * 3 OMR
            r.unitPrice += additionalCostFromArea; // Add this cost to the unit price
            r.totalPrice = r.unitPrice * r.qty; // Recalculate total price
        }
        // If the excess area condition (both height and width) is not met, check the old condition for width only
        else if (r.w > 100) { // If only width exceeds 1 meter (100 cm) (and height does not exceed 2.2 meters)
            const additionalWidthCm = r.w - 100;
            const additionalCost = additionalWidthCm * 0.3; // Cost of each additional 1 cm is 0.3 OMR
            r.unitPrice += additionalCost; // Add the additional cost to the unit price
            r.totalPrice = r.unitPrice * r.qty; // Recalculate total price
        }
    }
    // --- End of additional area calculation for WPC doors ---

    subtotal += r.totalPrice;
    let description = r.name;
    if (r.data && r.data.addons) {
        if(r.selectedAddons.curtain) description += ', with Curtain';
        if(r.selectedAddons.netType) description += `, with ${r.selectedAddons.netType} Net`;
    }
    if(r.itemNotes) description += ` (${r.itemNotes.replace(/\n/g, ', ')})`;

    tableHtml += `<tr>
                    <td style="padding: 10px; font-weight: bold;">${r.itemNotes || `B${index + 1}`}</td>
                    <td style="padding: 10px;">${formatNumber(r.h)}</td>
                    <td style="padding: 10px;">${formatNumber(r.w)}</td>
                    <td style="padding: 10px;">${formatNumber(r.h * r.w)}</td>
                    <td style="padding: 10px;">${r.qty}</td>
                    <td style="padding: 10px;"></td>
                    <td style="padding: 10px;"></td>
                    <td style="padding: 10px; background-color: #f2f2f2;">${formatNumber(r.unitPrice)}</td>
                    <td style="padding: 10px; background-color: #f2f2f2; font-weight: bold;">${formatNumber(r.totalPrice)}</td>
                    <td style="padding: 10px; text-align: left; white-space: pre-wrap;">${description}</td>
                  </tr>`;
});
tableHtml += `</tbody></table>`;
const commission = subtotal * 0.04;
// Ensure totalShippingCost is included in the grandTotal calculation
const grandTotal = subtotal + commission + totalShippingCost;
const summaryTable = `<table width="45%" align="right" style="border-collapse: collapse; font-family: Arial, sans-serif; margin-top: 25px; font-size: 15px;"><tbody>
            <tr><td style="padding: 14px; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">Subtotal</td><td style="padding: 14px; text-align: right; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(subtotal)} OMR</td></tr>
            <tr><td style="padding: 14px; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">Total Shipping Cost</td><td style="padding: 14px; text-align: right; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(totalShippingCost)} OMR</td></tr>
            <tr><td style="padding: 14px; font-weight: bold; border: 1px solid #ccc; font-size: 1.2em;">Office Commission (4%)</td><td style="padding: 14px; text-align: right; font-weight: bold; background-color: #f2f2f2; border: 1px solid #ccc; font-size: 1.2em;">${formatNumber(commission)} OMR</td></tr>
            <tr style="background-color: #4472C4; color: white;"><td style="padding: 16px; font-weight: bold; border: 1px solid #4472C4; font-size: 1.5em;">Grand Total</td><td style="padding: 16px; text-align: right; font-weight: bold; border: 1px solid #4472C4; font-size: 1.9em; vertical-align: middle;">${formatNumber(grandTotal)} OMR</td></tr>
            </tbody></table>`;
const footerNotesHtml = `<div style="clear: both; font-family: Arial, sans-serif; margin-top: 50px; text-align: center; border-top: 2px solid #4472C4; padding-top: 15px; font-size: 16px; color: #333;"><p style="margin: 5px 0;"><b>*Price includes delivery*</b></p><p style="margin: 5px 0;"><b>*Price does not include installation*</b></p></div>`;
const finalHtml = `<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'><head><meta charset='utf-8'><title>Quotation for - ${customerName}</title></head><body dir="ltr" style="padding: 20px;">`+ headerHtml + customerHtml + tableHtml + summaryTable + footerNotesHtml +`</body></html>`;
const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(finalHtml);
const fileDownload = document.createElement("a");
document.body.appendChild(fileDownload);
fileDownload.href = source;
fileDownload.download = `Quotation for - ${customerName}.doc`;
fileDownload.click();
document.body.removeChild(fileDownload);
```
