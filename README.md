<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kitchen Live Orders</title>
    <style>
        body { font-family: 'Segoe UI', system-ui, -apple-system, sans-serif; background: #f4f6f9; padding: 20px; color: #2d3748; display: flex; justify-content: center; margin: 0; }
        .container { width: 100%; max-width: 700px; }
        .header-row { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        h2 { margin: 0; color: #1a202c; font-size: 22px; }
        
  .btn-clear { background: #e53e3e; color: white; border: none; padding: 8px 14px; border-radius: 8px; font-weight: 600; cursor: pointer; font-size: 13px; transition: background 0.2s; }
        .btn-clear:hover { background: #c53030; }

  .order-card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.03); border: 1px solid #edf2f7; margin-bottom: 16px; }
        .order-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #edf2f7; padding-bottom: 10px; margin-bottom: 12px; }
        .item-row { display: flex; justify-content: space-between; font-size: 14px; margin: 6px 0; color: #4a5568; }
        .location-btn { display: inline-block; background: #3182ce; color: white; padding: 8px 14px; border-radius: 6px; text-decoration: none; font-size: 13px; font-weight: 600; margin-top: 10px; }
        .location-btn:hover { background: #2b6cb0; }
        .empty-state { text-align: center; color: #718096; padding: 40px; font-size: 15px; background: white; border-radius: 12px; border: 1px solid #edf2f7; }
    </style>
</head>
<body>

 <div class="container">
        <div class="header-row">
            <h2>🍳 Live Kitchen Orders (Firebase Cloud)</h2>
            <button class="btn-clear" onclick="clearAllOrders()">🗑️ Clear All Orders</button>
        </div>

   <div id="orderList"></div>
    </div>

<script>
    // Your Firebase Database URL
    const FIREBASE_URL = "https://test-d34cf-default-rtdb.firebaseio.com";

    async function loadOrders() {
        const container = document.getElementById('orderList');
        let orders = [];

        try {
            let res = await fetch(`${FIREBASE_URL}/orders.json`);
            let data = await res.json();
            if (Array.isArray(data)) orders = data;
        } catch (e) {
            container.innerHTML = '<div class="empty-state">Network connection error loading orders.</div>';
            return;
        }

        if (orders.length === 0) {
            container.innerHTML = '<div class="empty-state">No incoming orders right now. Waiting for customer checkouts from mobile devices...</div>';
            return;
        }

        let html = '';
        orders.forEach((o, index) => {
            let itemsHtml = '';
            if (o.items) {
                o.items.forEach(i => {
                    itemsHtml += `<div class="item-row"><span>${i.name} (x${i.quantity})</span><span>₹${i.price * i.quantity}</span></div>`;
                });
            }

            let locationButton = o.location ? `<div style="margin-top: 12px;"><a href="${o.location}" target="_blank" class="location-btn">📍 Open Delivery Location Pin ↗</a></div>` : '';

            html += `
                <div class="order-card">
                    <div class="order-header">
                        <div>
                            <strong style="font-size: 16px; color: #1a202c;">Order: ${o.id || ('#' + (index + 1))}</strong>
                            <div style="font-size: 12px; color: #718096; margin-top: 2px;">Customer: ${o.phone || 'Guest'} | Time: ${o.time || 'N/A'}</div>
                        </div>
                        <button style="background: #e53e3e; color: white; border: none; padding: 4px 8px; border-radius: 6px; font-size: 11px; cursor: pointer;" onclick="deleteSingleOrder(${index})">Remove</button>
                    </div>
                    <div>${itemsHtml}</div>
                    <hr style="border:0; border-top:1px dashed #cbd5e0; margin:12px 0;">
                    <div style="display: flex; justify-content: space-between; font-weight: 600; font-size: 14px; color: #1a202c;">
                        <span>Grand Total (incl. delivery):</span>
                        <span>₹${o.grandTotal || 500}</span>
                    </div>
                    ${locationButton}
                </div>
            `;
        });

        container.innerHTML = html;
    }

    async function deleteSingleOrder(index) {
        if (!confirm('Are you sure you want to remove this order?')) return;
        try {
            let res = await fetch(`${FIREBASE_URL}/orders.json`);
            let orders = await res.json();
            if (Array.isArray(orders)) {
                orders.splice(index, 1);
                await fetch(`${FIREBASE_URL}/orders.json`, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(orders)
                });
            }
            loadOrders();
        } catch (e) {
            alert('Failed to delete order.');
        }
    }

    async function clearAllOrders() {
        if (!confirm('Are you sure you want to clear all kitchen orders?')) return;
        try {
            await fetch(`${FIREBASE_URL}/orders.json`, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify([])
            });
            loadOrders();
        } catch (e) {
            alert('Failed to clear orders.');
        }
    }

    // Auto-refresh every 2 seconds to instantly pop up new orders from any device
    setInterval(loadOrders, 2000);
    loadOrders();
</script>
</body>
</html>
