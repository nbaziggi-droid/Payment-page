<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Bundle Payment</title>
  <script src="https://js.paystack.co/v1/inline.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: linear-gradient(135deg, #00c6ff, #0072ff);
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
    }
    .container {
      background: white;
      border-radius: 15px;
      padding: 30px;
      width: 350px;
      text-align: center;
      box-shadow: 0px 8px 20px rgba(0,0,0,0.2);
    }
    h2 {
      color: #0072ff;
      margin-bottom: 20px;
    }
    input, button {
      width: 100%;
      padding: 12px;
      margin: 10px 0;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-size: 16px;
    }
    button {
      background: #0072ff;
      color: white;
      font-weight: bold;
      border: none;
      cursor: pointer;
      transition: 0.3s;
    }
    button:hover {
      background: #0056cc;
    }
    .network {
      font-weight: bold;
      margin: 10px 0;
      color: green;
    }
  </style>
</head>
<body>
  <div class="container">
    <h2>Pay for Bundle</h2>
    
    <input type="tel" id="phone" placeholder="Enter Phone Number" oninput="detectNetwork()">
    <div id="network" class="network"></div>
    
    <input type="number" id="amount" placeholder="Enter Amount (Min ₵5)" min="5">
    
    <button onclick="initiatePayment()">Pay Now</button>
  </div>

  <script>
    let detectedNetwork = "";

    function detectNetwork() {
      let phone = document.getElementById("phone").value;
      let networkDisplay = document.getElementById("network");
      let prefix = phone.substring(0, 3);

      // MTN prefixes
      if (/^(053|024|055|025|059|054)/.test(prefix)) {
        detectedNetwork = "MTN";
      } 
      // Vodafone prefixes
      else if (/^(020|050)/.test(prefix)) {
        detectedNetwork = "Vodafone";
      } 
      // AirtelTigo prefixes
      else if (/^(027|057|026|056)/.test(prefix)) {
        detectedNetwork = "AirtelTigo";
      } 
      else {
        detectedNetwork = "";
      }

      networkDisplay.innerHTML = detectedNetwork ? "Detected Network: " + detectedNetwork : "";
    }

    function initiatePayment() {
      let phone = document.getElementById("phone").value;
      let amount = document.getElementById("amount").value;
      let merchantNumber = "0536622228";
      
      if (!phone || !detectedNetwork) {
        alert("Please enter a valid phone number to detect network.");
        return;
      }
      
      if (amount < 5) {
        alert("Minimum amount is ₵5");
        return;
      }

      alert("Detected Network: " + detectedNetwork);

      // Try USSD first
      let ussdCode = "";
      if (detectedNetwork === "MTN") {
        ussdCode = `*170*1*1*${merchantNumber}*${amount}#`;
      } else if (detectedNetwork === "Vodafone") {
        ussdCode = `*110*1*${merchantNumber}*${amount}#`;
      } else if (detectedNetwork === "AirtelTigo") {
        ussdCode = `*110*${merchantNumber}*${amount}#`;
      }

      if (ussdCode) {
        // Attempt to open dialer (only works on mobile)
        window.location.href = `tel:${ussdCode}`;

        // Fallback timer → If nothing happens in 3 seconds, use Paystack
        setTimeout(() => {
          fallbackToPaystack(phone, amount);
        }, 3000);
      } else {
        // Directly fallback if no USSD generated
        fallbackToPaystack(phone, amount);
      }
    }

    function fallbackToPaystack(phone, amount) {
      alert("Falling back to Paystack payment...");
      
      var handler = PaystackPop.setup({
        key: 'YOUR_PUBLIC_KEY', // replace with your Paystack public key
        email: 'customer@example.com', // replace dynamically with customer email
        amount: amount * 100, // in pesewas
        currency: "GHS",
        metadata: {
          custom_fields: [
            {
              display_name: "Phone Number",
              variable_name: "phone_number",
              value: phone
            },
            {
              display_name: "Detected Network",
              variable_name: "network",
              value: detectedNetwork
            }
          ]
        },
        callback: function(response){
          alert('Payment successful. Ref: ' + response.reference);
        },
        onClose: function(){
          alert('Payment window closed.');
        }
      });
      handler.openIframe();
    }
  </script>
</body>
</html>
