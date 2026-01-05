<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
    
    <title>Guardian Assistant Pro</title>
    
    <script src="https://cdn.jsdelivr.net/npm/@ericblade/quagga2/dist/quagga.min.js"></script>

    <style>
        :root { --g-green: #00754a; --g-orange: #ffc107; --danger: #d32f2f; --safe: #388e3c; --bg: #f8f9fa; }
        body { font-family: 'Segoe UI', Roboto, sans-serif; margin: 0; background: var(--bg); padding-bottom: 80px; -webkit-tap-highlight-color: transparent; }
        
        /* App UI Sections */
        .header { background: var(--g-green); color: white; padding: 45px 15px 15px; text-align: center; position: sticky; top: 0; z-index: 100; font-weight: bold; }
        .search-area { padding: 15px; background: white; position: sticky; top: 85px; z-index: 99; border-bottom: 1px solid #eee; }
        #searchInput { width: 100%; padding: 15px 20px; border: 2px solid #eee; border-radius: 30px; box-sizing: border-box; font-size: 16px; outline: none; }

        /* Category Scroll */
        .category-bar { display: flex; overflow-x: auto; background: white; padding: 10px; gap: 10px; scrollbar-width: none; }
        .category-bar::-webkit-scrollbar { display: none; }
        .cat-btn { padding: 10px 20px; background: #eee; border-radius: 25px; white-space: nowrap; border: none; font-size: 13px; font-weight: bold; color: #555; }
        .cat-btn.active { background: var(--g-green); color: white; }

        /* Product Cards */
        .list { padding: 15px; }
        .card { background: white; border-radius: 15px; padding: 15px; margin-bottom: 12px; display: flex; align-items: center; box-shadow: 0 2px 8px rgba(0,0,0,0.05); transition: 0.2s; }
        .info { flex-grow: 1; }
        .brand { font-size: 11px; color: var(--g-green); font-weight: 800; text-transform: uppercase; }
        .name { font-size: 15px; font-weight: 600; margin: 4px 0; }
        .price { color: #e60000; font-weight: 800; font-size: 18px; }
        .aisle { font-size: 11px; color: #888; margin-top: 5px; display: block; }

        /* Scanner Modal - Enhanced */
        #scanner-ui { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #000; z-index: 1000; }
        .scanner-container { width: 100%; height: 100%; position: relative; }
        #interactive { width: 100%; height: 100%; position: relative; }
        #interactive canvas { width: 100% !important; height: 100% !important; object-fit: cover; }
        .overlay-box { 
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 280px;
            height: 180px;
            border: 3px solid #00ff00;
            border-radius: 15px;
            box-shadow: 0 0 0 2000px rgba(0,0,0,0.7);
            z-index: 1001;
            animation: pulse 2s infinite;
        }
        @keyframes pulse {
            0% { border-color: #00ff00; }
            50% { border-color: #00cc00; }
            100% { border-color: #00ff00; }
        }
        .scanner-guide { 
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            text-align: center;
            z-index: 1002;
            width: 280px;
            padding-top: 200px;
            font-weight: bold;
            text-shadow: 0 1px 3px rgba(0,0,0,0.8);
        }

        /* Loading State */
        .loading { 
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.9);
            z-index: 3000;
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
        }
        .spinner { 
            width: 50px;
            height: 50px;
            border: 5px solid #f3f3f3;
            border-top: 5px solid var(--g-green);
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-bottom: 20px;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        /* Result Card Overlay */
        #result-card { 
            display: none; 
            position: fixed; 
            bottom: 0; 
            left: 0; 
            width: 100%; 
            height: 85vh; 
            background: white; 
            border-radius: 25px 25px 0 0; 
            padding: 25px; 
            box-sizing: border-box; 
            box-shadow: 0 -5px 25px rgba(0,0,0,0.2); 
            z-index: 2000; 
            overflow-y: auto;
            animation: slideUp 0.3s ease-out;
        }
        @keyframes slideUp {
            from { transform: translateY(100%); }
            to { transform: translateY(0); }
        }
        .status-badge { display: inline-block; padding: 6px 14px; border-radius: 15px; font-weight: bold; font-size: 11px; margin-bottom: 10px; }
        .status-danger { background: #ffebee; color: var(--danger); }
        .status-safe { background: #e8f5e9; color: var(--safe); }
        .highlight { color: var(--danger); text-decoration: underline; font-weight: bold; }
        .ingredient-list { 
            background: #f9f9f9; 
            padding: 15px; 
            border-radius: 10px; 
            margin-top: 15px;
            max-height: 200px;
            overflow-y: auto;
        }

        /* Navigation Bar */
        .nav { 
            position: fixed; 
            bottom: 0; 
            width: 100%; 
            background: white; 
            display: flex; 
            border-top: 1px solid #eee; 
            height: 75px; 
            align-items: center; 
            z-index: 500; 
            padding: 0 5px;
        }
        .nav-btn { 
            flex: 1; 
            text-align: center; 
            border: none; 
            background: none; 
            color: #888; 
            font-size: 11px; 
            font-weight: bold; 
            cursor: pointer; 
            padding: 10px 5px;
        }
        .scan-circle { 
            width: 65px; 
            height: 65px; 
            background: var(--g-green); 
            border-radius: 50%; 
            display: flex; 
            align-items: center; 
            justify-content: center; 
            color: white; 
            margin-top: -40px; 
            border: 8px solid var(--bg); 
            font-size: 24px;
            box-shadow: 0 4px 12px rgba(0,117,74,0.3);
        }
        
        /* Exit Button */
        .exit-btn {
            position: absolute;
            top: 20px;
            right: 20px;
            z-index: 1005;
            padding: 12px 24px;
            border-radius: 25px;
            border: none;
            background: rgba(255,255,255,0.9);
            font-weight: bold;
            font-size: 14px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.2);
        }
        
        /* Flash Toggle */
        .flash-btn {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 1005;
            padding: 12px 24px;
            border-radius: 25px;
            border: none;
            background: rgba(255,255,255,0.9);
            font-weight: bold;
            font-size: 14px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.2);
        }
    </style>
</head>
<body>

<div class="header">GUARDIAN HEALTH-GUARD PRO</div>

<div id="main-ui">
    <div class="search-area">
        <input type="text" id="searchInput" placeholder="Search brands (e.g. Wardah, Panadol)..." onkeyup="search()">
    </div>
    <div class="category-bar" id="catBar"></div>
    <div class="list" id="productList"></div>
</div>

<div id="scanner-ui">
    <div class="scanner-container">
        <div id="interactive" class="viewport"></div>
        <div class="overlay-box"></div>
        <div class="scanner-guide">Align barcode within the box</div>
        <button class="flash-btn" onclick="toggleFlash()">⚡ FLASH</button>
        <button class="exit-btn" onclick="stopScanner()">✕ EXIT</button>
    </div>
</div>

<div id="result-card">
    <div id="safety-badge" class="status-badge">Analyzing...</div>
    <h2 id="res-name" style="margin: 0; font-size: 20px;">Product Name</h2>
    <div style="font-size: 28px; color: #e60000; font-weight: bold; margin: 10px 0;" id="res-price">RM --.--</div>
    
    <div class="ingredient-list">
        <span style="font-weight: bold; font-size: 13px;">🔎 INGREDIENT ANALYSIS</span>
        <div id="warning-container" style="color:var(--danger); font-weight:bold; font-size:14px; margin: 10px 0;"></div>
        <div id="res-contents" style="font-size: 13px; color: #666; line-height: 1.4; border-top: 1px solid #ddd; padding-top: 10px;"></div>
    </div>
    
    <div style="margin-top: 20px; display: flex; gap: 10px;">
        <button onclick="document.getElementById('result-card').style.display='none'; startScanner();" style="flex: 1; background:var(--g-green); color:white; border:none; padding:15px; border-radius:10px; font-weight:bold;">SCAN NEXT</button>
        <button onclick="document.getElementById('result-card').style.display='none';" style="flex: 1; background:#eee; color:#333; border:none; padding:15px; border-radius:10px; font-weight:bold;">CLOSE</button>
    </div>
</div>

<div class="loading" id="loading">
    <div class="spinner"></div>
    <div id="loading-text">Analyzing product...</div>
</div>

<div class="nav">
    <button class="nav-btn" onclick="stopScanner()">🏠<br>HOME</button>
    <div class="nav-btn" onclick="startScanner()"><div class="scan-circle">🔍</div>SCAN</div>
    <button class="nav-btn" onclick="alert('Browse database active')">📂<br>ITEMS</button>
</div>

<script>
    // Enhanced red flags with more specific terms
    const redFlags = [
        "methylparaben", "propylparaben", "butylparaben", "ethylparaben",
        "alcohol denat", "sd alcohol", "isopropyl alcohol",
        "sodium lauryl sulfate", "sls", "sodium laureth sulfate", "sles",
        "fragrance", "parfum", "perfume",
        "phthalate", "dibutyl phthalate", "dep", "dehp",
        "dimethicone", "cyclomethicone", "simethicone"
    ];
    
    // Database
    const rawBrands = {
        "Skincare": ["Cetaphil", "Hada Labo", "Bio-Essence", "Wardah", "Loreal", "Eucerin", "Simple", "Garnier", "Neutrogena", "Sunsilk", "Olay", "Cosrx", "Nivea", "Clinelle", "Aiken"],
        "Health": ["Panadol", "Gaviscon", "Blackmores", "Hurix's", "Flavettes", "Brands", "Eno", "Tiger Balm", "Vicks", "Strepsils", "Eye Mo", "Betadine", "Dettol Antiseptic", "Woods", "Cap Ibu & Anak"],
        "Personal Care": ["Dettol Shower", "Colgate", "Pantene", "Dove", "Lifebuoy", "Rexona", "Sunplay", "Sensodyne", "Oral-B", "Shokubutsu", "May", "Ginvera", "Kotex", "Libresse", "Carefree"],
        "Cosmetics": ["Maybelline", "Silkygirl", "Revlon", "Wardah Colorfit", "In2It", "Kate", "Peripera", "Essence", "Catrice", "Elianto"],
        "Baby": ["Johnson's", "Huggies", "MamyPoko", "Pureen", "Pigeon", "Cetaphil Baby", "Sebamed", "Biolane"]
    };

    let fullDb = [];
    Object.keys(rawBrands).forEach(cat => {
        rawBrands[cat].forEach(brand => {
            for(let i=1; i<=3; i++) {
                fullDb.push({
                    b: brand,
                    n: `${brand} ${cat} Series ${i}`,
                    p: (Math.random() * 85 + 5).toFixed(2),
                    c: cat,
                    a: `Aisle ${Math.floor(Math.random() * 10) + 1}`
                });
            }
        });
    });

    // Scanner Variables
    let scannerActive = false;
    let flashOn = false;
    
    // --- Search & UI Logic ---
    function render(data) {
        document.getElementById('productList').innerHTML = data.map(i => `
            <div class="card">
                <div class="info">
                    <div class="brand">${i.b}</div>
                    <div class="name">${i.n}</div>
                    <p class="price">RM ${i.p}</p>
                    <span class="aisle">📍 ${i.c} | ${i.a}</span>
                </div>
            </div>
        `).join('');
    }

    function search() {
        const q = document.getElementById('searchInput').value.toLowerCase();
        render(fullDb.filter(i => i.n.toLowerCase().includes(q) || i.b.toLowerCase().includes(q)));
    }

    function filter(cat, btn) {
        document.querySelectorAll('.cat-btn').forEach(b => b.classList.remove('active'));
        btn.classList.add('active');
        render(cat === "All" ? fullDb : fullDb.filter(i => i.c === cat));
    }

    // --- Enhanced Scanner Logic ---
    function startScanner() {
        document.getElementById('main-ui').style.display = 'none';
        document.getElementById('scanner-ui').style.display = 'block';
        document.getElementById('result-card').style.display = 'none';
        
        // Configure scanner for high resolution
        Quagga.init({
            inputStream: {
                name: "Live",
                type: "LiveStream",
                target: document.querySelector('#interactive'),
                constraints: {
                    facingMode: "environment",
                    width: { min: 640, ideal: 1280, max: 1920 },
                    height: { min: 480, ideal: 720, max: 1080 },
                    aspectRatio: { min: 1, max: 2 }
                },
                area: { // Define scanning area (center 60%)
                    top: "20%",
                    right: "20%",
                    left: "20%",
                    bottom: "20%"
                }
            },
            decoder: {
                readers: [
                    "ean_reader",
                    "ean_8_reader",
                    "upc_reader",
                    "upc_e_reader",
                    "code_128_reader",
                    "code_39_reader"
                ],
                multiple: false
            },
            locate: true,
            numOfWorkers: 4,
            frequency: 10, // Check 10 times per second
            debug: {
                drawBoundingBox: true,
                showFrequency: false,
                drawScanline: true,
                showPattern: true
            }
        }, function(err) {
            if (err) {
                console.error("Scanner initialization failed:", err);
                alert("Camera access denied or scanner failed to initialize. Please allow camera permissions.");
                stopScanner();
                return;
            }
            console.log("Scanner initialized successfully");
            Quagga.start();
            scannerActive = true;
        });

        Quagga.onDetected(async function(result) {
            if (!result || !result.codeResult) return;
            
            const code = result.codeResult.code;
            console.log("Barcode detected:", code);
            
            Quagga.offDetected();
            Quagga.stop();
            scannerActive = false;
            
            // Show loading
            document.getElementById('loading').style.display = 'flex';
            document.getElementById('loading-text').innerText = 'Analyzing product...';
            
            await analyzeProduct(code);
            
            // Hide loading
            document.getElementById('loading').style.display = 'none';
        });

        Quagga.onProcessed(function(result) {
            if (!result) return;
            
            const drawingCtx = Quagga.canvas.ctx.overlay;
            const drawingCanvas = Quagga.canvas.dom.overlay;
            
            if (result.boxes) {
                drawingCtx.clearRect(0, 0, parseInt(drawingCanvas.getAttribute("width")), parseInt(drawingCanvas.getAttribute("height")));
                result.boxes.filter(function(box) {
                    return box !== result.box;
                }).forEach(function(box) {
                    Quagga.ImageDebug.drawPath(box, {x: 0, y: 1}, drawingCtx, {color: "green", lineWidth: 2});
                });
            }
            
            if (result.box) {
                Quagga.ImageDebug.drawPath(result.box, {x: 0, y: 1}, drawingCtx, {color: "#00F", lineWidth: 2});
            }
            
            if (result.codeResult && result.codeResult.code) {
                Quagga.ImageDebug.drawPath(result.line, {x: 'x', y: 'y'}, drawingCtx, {color: 'red', lineWidth: 3});
            }
        });
    }

    function toggleFlash() {
        // This is a simplified flash toggle
        // In a real app, you would use the MediaTrackConstraints
        flashOn = !flashOn;
        alert(flashOn ? "Flash turned on" : "Flash turned off");
        // Note: Actual flash control requires specific device API access
    }

    async function analyzeProduct(barcode) {
        document.getElementById('scanner-ui').style.display = 'none';
        document.getElementById('result-card').style.display = 'block';
        
        try {
            // Show product info from our database if available
            const localProduct = fullDb.find(p => p.b.toLowerCase().includes(barcode.substring(0, 3).toLowerCase()));
            
            // Try OpenFoodFacts API
            const response = await fetch(`https://world.openfoodfacts.org/api/v0/product/${barcode}.json`);
            const data = await response.json();
            
            let productName = "Scanned Product";
            let ingredients = "No ingredient information available.";
            let price = "RM " + (Math.random() * 40 + 10).toFixed(2);
            
            if (data.status === 1 && data.product) {
                const p = data.product;
                productName = p.product_name || p.generic_name || "Unknown Product";
                ingredients = p.ingredients_text || "Ingredients not specified.";
                
                if (localProduct) {
                    price = "RM " + localProduct.p;
                }
            } else if (localProduct) {
                productName = localProduct.n;
                ingredients = `Standard ${localProduct.b} formula. Check packaging for full ingredient list.`;
                price = "RM " + localProduct.p;
            }
            
            document.getElementById('res-name').innerText = productName;
            document.getElementById('res-price').innerText = price;
            
            // Enhanced ingredient analysis
            let foundFlags = [];
            let highlightedText = ingredients;
            
            redFlags.forEach(flag => {
                const regex = new RegExp(`\\b${flag}\\b`, "gi");
                if (regex.test(ingredients)) {
                    foundFlags.push(flag.toUpperCase());
                    highlightedText = highlightedText.replace(regex, `<span class="highlight">${flag}</span>`);
                }
            });
            
            // Check for common irritants patterns
            const irritantPatterns = [
                { pattern: /\bparaben\b/gi, name: "PARABEN" },
                { pattern: /\balcohol\s*(denat|sd)?\b/gi, name: "ALCOHOL" },
                { pattern: /\bsulfate\b/gi, name: "SULFATE" }
            ];
            
            irritantPatterns.forEach(pattern => {
                if (pattern.pattern.test(ingredients) && !foundFlags.some(f => f.includes(pattern.name))) {
                    foundFlags.push(pattern.name);
                }
            });
            
            const badge = document.getElementById('safety-badge');
            const warning = document.getElementById('warning-container');
            
            if (foundFlags.length > 2) {
                badge.className = "status-badge status-danger";
                badge.innerText = "⚠️ HIGH RISK - MULTIPLE IRRITANTS";
                warning.innerText = `Detected ${foundFlags.length} potential irritants: ${foundFlags.slice(0, 3).join(", ")}${foundFlags.length > 3 ? '...' : ''}`;
            } else if (foundFlags.length > 0) {
                badge.className = "status-badge status-danger";
                badge.innerText = "⚠️ CONTAINS POTENTIAL IRRITANTS";
                warning.innerText = "Detected: " + foundFlags.join(", ");
            } else {
                badge.className = "status-badge status-safe";
                badge.innerText = "✅ LOW RISK FORMULATION";
                warning.innerText = "No common irritants detected. Always patch test new products.";
            }
            
            document.getElementById('res-contents').innerHTML = highlightedText || "No ingredient data available.";
            
        } catch (error) {
            console.error("Analysis error:", error);
            document.getElementById('res-name').innerText = "Scan Result";
            document.getElementById('res-price').innerText = "RM --.--";
            document.getElementById('safety-badge').className = "status-badge status-danger";
            document.getElementById('safety-badge').innerText = "⚠️ ANALYSIS ERROR";
            document.getElementById('warning-container').innerText = "Unable to retrieve product data. Please try again.";
            document.getElementById('res-contents').innerText = "Network or database error occurred.";
        }
    }

    function stopScanner() {
        if (scannerActive) {
            Quagga.stop();
            scannerActive = false;
        }
        document.getElementById('scanner-ui').style.display = 'none';
        document.getElementById('main-ui').style.display = 'block';
        document.getElementById('result-card').style.display = 'none';
        document.getElementById('loading').style.display = 'none';
        
        // Clean up
        const interactive = document.getElementById('interactive');
        if (interactive) {
            interactive.innerHTML = '';
        }
    }

    // Initialize UI
    window.addEventListener('load', function() {
        const categories = ["All", ...Object.keys(rawBrands)];
        document.getElementById('catBar').innerHTML = categories.map(c => 
            `<button class="cat-btn ${c==='All'?'active':''}" onclick="filter('${c}', this)">${c}</button>`
        ).join('');
        render(fullDb);
    });

    // Handle page visibility changes
    document.addEventListener('visibilitychange', function() {
        if (document.hidden && scannerActive) {
            stopScanner();
        }
    });
</script>
</body>
</html>
