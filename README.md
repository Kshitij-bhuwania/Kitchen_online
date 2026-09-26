<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Kitchen Live Orders</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: #edf2f7; padding: 30px; }
        .container { max-width: 800px; margin: 0 auto; }
        .order-card { background: white; border-radius: 10px; padding: 20px; margin-bottom: 20px; box-shadow: 0 4px 10px rgba(0,0,0,0.05); border-left: 6px solid #2ed573; }
        .order-header { display: flex; justify-content: space-between; font-weight: bold; font-size: 16px; margin-bottom: 10px; color: #333; }
        .order-details { margin: 10px 0; color: #555; font-size: 14px; }
        ul { padding-left: 20px; margin: 5px 0; }
        .summary-row { display: flex; justify-content: space-between; border-top: 1px solid #eee; margin-top: 15px; padding-top: 10px; font-weight: bold; }
        .map-link { color: #1e90ff; font-weight: bold; text-decoration: underline; }
        .back-btn { background: #333; color: white; padding: 10px 15px; border: none; border-radius: 6px; cursor: pointer; margin-bottom: 20px; }
    </style>
</head>
<body>

    <div class="container">
        <button class="back-btn" onclick="window.location.href='menu.html'">← Back to Menu</button>
        <h1>Kitchen Live Orders Feed</h1>
        <div id="ordersContainer"></div>
    </div>

<script>
    function renderOrders() {
        const orders = JSON.parse(localStorage.getItem('liveOrders') || '[]');
        const container = document.getElementById('ordersContainer');
        container.innerHTML = '';

        if (orders.length === 0) {
            container.innerHTML = '<p style="color: #666;">No orders checked out yet.</p>';
            return;
        }

        orders.forEach(order => {
            let itemsHtml = order.items.map(i => `<li>${i.name} (x${i.quantity}) — ₹${i.price * i.quantity}</li>`).join('');

            // Check if location is a link or standard text
            let locationDisplay = order.location.startsWith('http') 
                ? `<a href="${order.location}" target="_blank" class="map-link">Open Google Maps Pin ↗</a>` 
                : order.location;

            container.innerHTML += `
                <div class="order-card">
                    <div class="order-header">
                        <span>#️⃣ Order Serial No: #${order.serialNumber}</span>
                        <span>📱 Phone: ${order.phone}</span>
                    </div>
                    <div class="order-details">
                        <p><strong>Location:</strong> ${locationDisplay}</p>
                        <p><strong>Items:</strong></p>
                        <ul>${itemsHtml}</ul>
                    </div>
                    <div class="summary-row">
                        <span>Total Items Count: ${order.totalItemsCount}</span>
                        <span>Grand Value: ₹${order.grandTotal} (Inc. ₹${order.deliveryCharge} delivery)</span>
                    </div>
                    <div style="font-size: 11px; color: #999; margin-top: 8px; text-align: right;">Time: ${order.timestamp}</div>
                </div>
            `;
        });
    }

    renderOrders();
    setInterval(renderOrders, 2000); // Live sync polling
</script>
</body>
</html>
