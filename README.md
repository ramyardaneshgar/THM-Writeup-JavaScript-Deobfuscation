# JavaScript-Deobfuscation
JavaScript Deobfuscation Techniques,  reverse engineering HTTP request flows, and decoded payloads using cURL, base64, and xxd.

**Developed By**: Ramyar Daneshgar  

---

## **Overview**

This module focused on analyzing obfuscated JavaScript code, interacting with HTTP endpoints, and decoding hidden data. Such techniques are crucial for detecting and analyzing malicious scripts, reverse-engineering web-based threats, and understanding how threat actors obfuscate payloads to evade detection. This writeup outlines my systematic approach to uncovering hidden flags, using tools like **cURL**, **base64**, **xxd**, and **JSConsole** for static and dynamic analysis.

---

## **1. Source Code Analysis**

**Question**: *“What is the flag in the HTML source code?”*  

### **Approach:**  
After spawning the target machine, I navigated to the target's root page and performed a static analysis of the HTML source code.  

1. I used the browser’s "View Page Source" option to inspect the structure.  
2. Scanning through the code manually, I identified an exposed **HTML comment** on **line 48** containing the hidden flag.

### **Rationale:**  
Static code analysis often reveals sensitive information left behind by developers, such as debug comments, API keys, or credentials. In a real-world scenario, this could be exploited during reconnaissance or as part of an attack chain.

---

## **2. Deobfuscating Obfuscated JavaScript**

**Question**: *“Deobfuscate the `secret.js` file to retrieve the flag.”*  

### **Approach:**  
1. In the HTML source code, I identified an external JavaScript file named `secret.js`.  
2. I downloaded the file and inspected its content, which contained obfuscated code using packed `eval` functions.  
3. I copied the script and used **UnPacker**, a tool specifically designed to reverse common obfuscation techniques like **eval** and string concatenation.  
4. After unpacking, the flag was clearly visible in the deobfuscated output.  

### **Rationale:**  
JavaScript obfuscation techniques like `eval()` packing and string manipulation are commonly used by threat actors to hide malicious payloads or exfiltration scripts. Tools like **UnPacker** automate reversing obfuscation for faster analysis.

---

## **3. Interacting with HTTP Endpoints**

**Question**: *“Send a POST request to `/serial.php`. What is the response?”*  

### **Approach:**  
1. I used **cURL** to interact with the `/serial.php` endpoint:  
   ```bash
   curl -w "\n" -s -X POST http://STMIP:STMPO/serial.php
   ```  
   - `-X POST`: Simulates an HTTP POST request.  
   - `-s`: Silent mode to suppress extraneous output.  
   - `-w "\n"`: Ensures a clean output format.  
2. The response returned an encoded string, likely obfuscated or encoded to conceal the flag.

### **Rationale:**  
Manual HTTP interaction with tools like **cURL** mimics legitimate client behavior. This skill is vital for API testing, interacting with malicious servers, or probing endpoints during pentesting. POST requests simulate user submissions or form actions.  

---

## **4. Decoding Encoded Data**

**Question**: *“Decode the string and send it back to the server.”*  

### **Approach:**  
1. The returned string `N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz` was analyzed using **Cipher Identifier**, which confirmed it as **Base64**.  
2. I decoded it locally using the `base64` command:  
   ```bash
   echo 'N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz' | base64 -d
   ```  
   - Decoded Output: `7h15_15_a_s3cr37_m3554g3`.  
3. I sent the decoded flag back to `/serial.php` via cURL:  
   ```bash
   curl -w "\n" -s -X POST "http://STMIP:STMPO/serial.php" -d "serial=7h15_15_a_s3cr37_m3554g3"
   ```  

### **Rationale:**  
Base64 encoding is a widely used method to obfuscate plain text data, often for transmitting information in HTTP requests, emails, or scripts. Identifying and decoding such data is a core skill in malware analysis and threat hunting.

---

## **5. Skills Assessment: JavaScript Dynamic Analysis**

### **Question 1 & 2: Running Obfuscated JavaScript**  
1. I identified `api.min.js` in the webpage source code.  
2. The JavaScript was packed with an **eval-based obfuscation** technique.  
3. I ran the script dynamically in **JSConsole** to evaluate its behavior.  
   - Output: The script concatenated strings dynamically to reveal the hidden flag.  

### **Question 3: Deobfuscating JavaScript Code**  
1. I copied the obfuscated `api.min.js` script into **JS Nice** to clean up and analyze its functionality.  
2. After deobfuscation, I observed that the script explicitly declared a `flag` variable.  
3. By running:  
   ```javascript
   console.log(flag);
   ```  
   I extracted the flag.

---

### **Question 4 & 5: Replicating Functionality and Hexadecimal Decoding**  
1. The deobfuscated script sent a **POST request** to `/keys.php`. To replicate this, I executed:  
   ```bash
   curl -w "\n" -s http://STMIP:STMPO/keys.php -X POST
   ```  
   - Response: A **hexadecimal-encoded string** `4150495f70336e5f37333537316e365f31355f66756e`.  
2. I decoded the string using `xxd` to convert hex to plain text:  
   ```bash
   echo 4150495f70336e5f37333537316e365f31355f66756e | xxd -p -r
   ```  
   - Output: `API_p3n_73571n6_15_fun`.  
3. Finally, I sent this decoded key as a parameter to `/keys.php`:  
   ```bash
   curl -w "\n" -s http://STMIP:STMPO/keys.php -X POST -d 'key=API_p3n_73571n6_15_fun'
   ```  
   - Response: The final flag.

---

## **Key Takeaways**

1. **JavaScript Obfuscation**: Techniques like `eval()` packing and string concatenation are common in malicious scripts. Using tools like **UnPacker** and **JS Nice** enables efficient analysis.  
2. **HTTP Endpoint Interaction**: cURL is a critical tool for interacting with endpoints, simulating requests, and testing behavior—skills applicable in penetration testing and API security assessments.  
3. **Encoding & Decoding**: Recognizing encoding schemes (Base64, Hex) and decoding them using native tools like `base64` and `xxd` is essential for analyzing obfuscated or exfiltrated data.  
4. **Dynamic Analysis**: Executing and analyzing JavaScript in controlled environments (e.g., JSConsole) reveals hidden behaviors, critical for detecting malicious functionality.  
5. **Logic**: Replicating script behavior and interacting with backend endpoints demonstrates a deep understanding of web-based attack surfaces and logic exploitation.

---
