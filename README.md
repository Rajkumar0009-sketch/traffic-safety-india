<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>India Traffic Safety Dashboard - Data Analytics Project</title>
    <!-- GitHub Meta Tags -->
    <meta name="description" content="Interactive dashboard analyzing traffic accidents in India using data analytics, Power BI, SQL and machine learning">
    <meta name="keywords" content="data analytics, traffic safety, India, dashboard, SQL, Power BI, machine learning">
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css">
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f8f9fa;
        }
        .card {
            border-radius: 10px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            transition: transform 0.3s ease;
        }
        .card:hover {
            transform: translateY(-5px);
        }
        #map {
            height: 500px;
            border-radius: 10px;
            z-index: 1;
        }
        .navbar-brand {
            font-weight: 700;
        }
        .sidebar {
            background-color: #343a40;
            color: white;
            height: 100vh;
            position: fixed;
        }
        .dashboard-card-header {
            border-bottom: 1px solid rgba(0, 0, 0, 0.1);
            padding: 15px 20px;
            font-weight: 600;
        }
        .risk-indicator {
            width: 20px;
            height: 20px;
            border-radius: 50%;
            display: inline-block;
        }
        .risk-low {
            background-color: #28a745;
        }
        .risk-medium {
            background-color: #ffc107;
        }
        .risk-high {
            background-color: #dc3545;
        }
        .data-table {
            font-size: 0.85rem;
        }
        .info-box {
            border-left: 4px solid;
            padding: 10px;
            margin-bottom: 15px;
            background-color: white;
        }
        .real-time-alert {
            animation: pulse 2s infinite;
        }
        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(220, 53, 69, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(220, 53, 69, 0); }
            100% { box-shadow: 0 0 0 0 rgba(220, 53, 69, 0); }
        }
    </style>
</head>
<body>
    <div class="container-fluid">
        <div class="row">
            <!-- Sidebar -->
            <div class="col-md-2 sidebar d-none d-md-block">
                <div class="p-3">
                    <h4 class="text-center mb-4">Traffic Safety Dashboard</h4>
                    <hr>
                    <ul class="nav flex-column">
                        <li class="nav-item">
                            <a class="nav-link active" href="#dashboard">
                                <i class="fas fa-tachometer-alt me-2"></i> Dashboard
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#heatmaps">
                                <i class="fas fa-map-marked-alt me-2"></i> Heatmaps
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#analytics">
                                <i class="fas fa-chart-line me-2"></i> Analytics
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#predictions">
                                <i class="fas fa-robot me-2"></i> Predictions
                            </a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link" href="#reports">
                                <i class="fas fa-file-pdf me-2"></i> Reports
                            </a>
                        </li>
                    </ul>
                    <hr>
                    <div class="mt-4">
                        <h6>Data Sources</h6>
                        <ul class="list-unstyled">
                            <li><i class="fas fa-check-circle text-success me-2"></i> MoRTH</li>
                            <li><i class="fas fa-check-circle text-success me-2"></i> data.gov.in</li>
                            <li><i class="fas fa-check-circle text-success me-2"></i> WHO</li>
                            <li><i class="fas fa-sync-alt text-warning me-2"></i> Real-time IoT</li>
                        </ul>
                    </div>
                </div>
            </div>

            <!-- Main Content -->
            <div class="col-md-10 ms-auto">
                <nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm">
                    <div class="container-fluid">
                        <a class="navbar-brand" href="#">
                            <i class="fas fa-car-crash text-danger me-2"></i>
                            Indian Road Safety Analytics
                        </a>
                        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
                            <span class="navbar-toggler-icon"></span>
                        </button>
                        <div class="collapse navbar-collapse" id="navbarNav">
                            <ul class="navbar-nav ms-auto">
                                <li class="nav-item">
                                    <div class="input-group">
                                        <select class="form-select" id="stateSelector">
                                            <option selected>All India</option>
                                            <option>Maharashtra</option>
                                            <option>Uttar Pradesh</option>
                                            <option>Karnataka</option>
                                            <option>Tamil Nadu</option>
                                            <option>Delhi</option>
                                        </select>
                                        <button class="btn btn-primary" type="button">
                                            <i class="fas fa-filter"></i> Filter
                                        </button>
                                    </div>
                                </li>
                                <li class="nav-item ms-2">
                                    <button class="btn btn-success" id="realtimeToggle">
                                        <i class="fas fa-bolt"></i> Live Mode
                                    </button>
                                </li>
                            </ul>
                        </div>
                    </div>
                </nav>

                <div class="p-4" id="dashboard">
                    <!-- Alerts Section -->
                    <div class="row mb-4">
                        <div class="col-md-12">
                            <div class="alert alert-danger real-time-alert d-flex justify-content-between align-items-center">
                                <div>
                                    <strong><i class="fas fa-exclamation-triangle me-2"></i> HIGH RISK ALERT:</strong> 
                                    Increased accidents reported on NH48 between Delhi and Jaipur (12% above predicted)
                                </div>
                                <button class="btn btn-sm btn-outline-light">View Details</button>
                            </div>
                        </div>
                    </div>

                    <!-- Key Metrics Row -->
                    <div class="row">
                        <!-- Annual Deaths -->
                        <div class="col-md-3 mb-4">
                            <div class="card h-100">
                                <div class="dashboard-card-header d-flex justify-content-between align-items-center">
                                    <span>Annual Deaths</span>
                                    <i class="fas fa-skull text-danger"></i>
                                </div>
                                <div class="card-body text-center">
                                    <h1 class="display-4 text-danger">153,972</h1>
                                    <div class="mt-2">
                                        <span class="text-danger me-2"><i class="fas fa-arrow-up"></i> 5.2%</span>
                                        <span class="text-muted">from last year</span>
                                    </div>
                                    <div class="progress mt-2" style="height: 6px;">
                                        <div class="progress-bar bg-danger" role="progressbar" style="width: 72%"></div>
                                    </div>
                                    <small class="text-muted">India accounts for 11% of global road deaths</small>
                                </div>
                            </div>
                        </div>

                        <!-- Primary Causes -->
                        <div class="col-md-3 mb-4">
                            <div class="card h-100">
                                <div class="dashboard-card-header d-flex justify-content-between align-items-center">
                                    <span>Primary Causes</span>
                                    <i class="fas fa-search-plus text-info"></i>
                                </div>
                                <div class="card-body">
                                    <div class="d-flex mb-2">
                                        <div class="risk-indicator risk-high me-2"></div>
                                        <div class="flex-grow-1">Overspeeding</div>
                                        <div class="font-weight-bold">48%</div>
                                    </div>
                                    <div class="d-flex mb-2">
                                        <div class="risk-indicator risk-high me-2"></div>
                                        <div class="flex-grow-1">Drunken Driving</div>
                                        <div class="font-weight-bold">23%</div>
                                    </div>
                                    <div class="d-flex mb-2">
                                        <div class="risk-indicator risk-medium me-2"></div>
                                        <div class="flex-grow-1">Vehicle Defects</div>
                                        <div class="font-weight-bold">12%</div>
                                    </div>
                                    <div class="d-flex">
                                        <div class="risk-indicator risk-medium me-2"></div>
                                        <div class="flex-grow-1">Road Conditions</div>
                                        <div class="font-weight-bold">17%</div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- High Risk Areas -->
                        <div class="col-md-3 mb-4">
                            <div class="card h-100">
                                <div class="dashboard-card-header d-flex justify-content-between align-items-center">
                                    <span>High Risk Areas</span>
                                    <i class="fas fa-map-marker-alt text-warning"></i>
                                </div>
                                <div class="card-body">
                                    <div class="mb-2">
                                        <strong>1. NH44 (Kanyakumari to Srinagar)</strong>
                                        <div class="text-warning">
                                            <i class="fas fa-car-crash"></i> 8,742 deaths/year
                                        </div>
                                    </div>
                                    <div class="mb-2">
                                        <strong>2. NH48 (Delhi to Chennai)</strong>
                                        <div class="text-warning">
                                            <i class="fas fa-car-crash"></i> 6,512 deaths/year
                                        </div>
                                    </div>
                                    <div class="mb-2">
                                        <strong>3. NH27 (Porbandar to Silchar)</strong>
                                        <div class="text-warning">
                                            <i class="fas fa-car-crash"></i> 5,223 deaths/year
                                        </div>
                                    </div>
                                    <small class="text-muted">Top 10 highways account for 42% of deaths</small>
                                </div>
                            </div>
                        </div>

                        <!-- Prediction Accuracy -->
                        <div class="col-md-3 mb-4">
                            <div class="card h-100">
                                <div class="dashboard-card-header d-flex justify-content-between align-items-center">
                                    <span>Prediction Accuracy</span>
                                    <i class="fas fa-brain text-success"></i>
                                </div>
                                <div class="card-body text-center">
                                    <div class="gauge-container">
                                        <canvas id="accuracyGauge" width="150" height="100"></canvas>
                                    </div>
                                    <div class="mt-3">
                                        <span class="badge bg-success">83.7% Accuracy</span>
                                        <p class="small mt-2">ML model predicting fatal accidents</p>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Map and Time Charts Row -->
                    <div class="row mb-4">
                        <!-- Map Visualization -->
                        <div class="col-md-8 mb-4">
                            <div class="card h-100">
                                <div class="dashboard-card-header d-flex justify-content-between align-items-center">
                                    <span>Accident Heatmap - India</span>
                                    <div>
                                        <button class="btn btn-sm btn-outline-primary me-1 active" data-layer="heat">Heatmap</button>
                                        <button class="btn btn-sm btn-outline-secondary me-1" data-layer="cluster">Clusters</button>
                                        <button class="btn btn-sm btn-outline-info" data-layer="predictive">Predictive</button>
                                    </div>
                                </div>
                                <div class="card-body p-0">
                                    <div id="map"></div>
                                </div>
                            </div>
                        </div>

                        <!-- Time Patterns -->
                        <div class="col-md-4 mb-4">
                            <div class="card h-100">
                                <div class="dashboard-card-header">
                                    Accident Time Patterns
                                </div>
                                <div class="card-body">
                                    <canvas id="timeChart"></canvas>
                                    <div class="mt-3">
                                        <div class="alert alert-warning">
                                            <i class="fas fa-clock me-1"></i> Peak hours: 6-10 PM (38% of accidents)
                                        </div>
                                        <div class="alert alert-dark">
                                            <i class="fas fa-moon me-1"></i> Night accidents are 2.7x more fatal
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Detailed Analytics Row -->
                    <div class="row">
                        <!-- Vehicle Type Analysis -->
                        <div class="col-md-6 mb-4">
                            <div class="card h-100">
                                <div class="dashboard-card-header">
                                    Vehicle Type Analysis
                                </div>
                                <div class="card-body">
                                    <canvas id="vehicleChart"></canvas>
                                    <div class="row mt-3">
                                        <div class="col-md-6">
                                            <div class="info-box border-primary">
                                                <h6>Most Dangerous</h6>
                                                <p class="mb-0">Two-wheelers: 37% of deaths</p>
                                            </div>
                                        </div>
                                        <div class="col-md-6">
                                            <div class="info-box border-info">
                                                <h6>Increasing Risk</h6>
                                                <p class="mb-0">Delivery vans: +23% YoY</p>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- Demographic Analysis -->
                        <div class="col-md-6 mb-4">
                            <div class="card h-100">
                                <div class="dashboard-card-header">
                                    Victim Demographics
                                </div>
                                <div class="card-body">
                                    <div class="row">
                                        <div class="col-md-6">
                                            <canvas id="ageChart"></canvas>
                                        </div>
                                        <div class="col-md-6">
                                            <canvas id="genderChart"></canvas>
                                        </div>
                                    </div>
                                    <div class="row mt-3">
                                        <div class="col-md-12">
                                            <div class="alert alert-info mt-2">
                                                <strong>Key Insight:</strong> 67% of victims are pedestrians, cyclists or two-wheeler riders
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Real-time Data Section -->
                    <div class="row mb-4">
                        <div class="col-md-12">
                            <div class="card">
                                <div class="dashboard-card-header d-flex justify-content-between align-items-center">
                                    <span><i class="fas fa-bolt me-2"></i> Real-time Incident Feed</span>
                                    <button class="btn btn-sm btn-success">
                                        <i class="fas fa-sync-alt me-1"></i> Refresh
                                    </button>
                                </div>
                                <div class="card-body p-0">
                                    <div class="table-responsive">
                                        <table class="table table-hover mb-0 data-table">
                                            <thead class="table-light">
                                                <tr>
                                                    <th>Time</th>
                                                    <th>Location</th>
                                                    <th>Type</th>
                                                    <th>Severity</th>
                                                    <th>Probable Cause</th>
                                                    <th>Action</th>
                                                </tr>
                                            </thead>
                                            <tbody id="realtimeTable">
                                                <!-- Data will be populated by JavaScript -->
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- ML Predictions Section -->
                    <div class="row mb-4">
                        <div class="col-md-12">
                            <div class="card">
                                <div class="dashboard-card-header">
                                    Machine Learning Predictions for Upcoming Diwali Weekend
                                </div>
                                <div class="card-body">
                                    <div class="alert alert-warning">
                                        <strong>Forecast:</strong> Expected 18-22% increase in accidents compared to normal weekend
                                    </div>
                                    <div class="row">
                                        <div class="col-md-6">
                                            <canvas id="predictionChart"></canvas>
                                        </div>
                                        <div class="col-md-6">
                                            <h5>Top Risk Factors</h5>
                                            <div class="mb-2">
                                                <div class="d-flex justify-content-between mb-1">
                                                    <span>Drunken Driving</span>
                                                    <span>42% contribution</span>
                                                </div>
                                                <div class="progress" style="height: 8px;">
                                                    <div class="progress-bar bg-danger" role="progressbar" style="width: 42%"></div>
                                                </div>
                                            </div>
                                            <div class="mb-2">
                                                <div class="d-flex justify-content-between mb-1">
                                                    <span>Nighttime Driving</span>
                                                    <span>38% contribution</span>
                                                </div>
                                                <div class="progress" style="height: 8px;">
                                                    <div class="progress-bar bg-warning" role="progressbar" style="width: 38%"></div>
                                                </div>
                                            </div>
                                            <div class="mb-2">
                                                <div class="d-flex justify-content-between mb-1">
                                                    <span>Overcrowded Vehicles</span>
                                                    <span>20% contribution</span>
                                                </div>
                                                <div class="progress" style="height: 8px;">
                                                    <div class="progress-bar bg-info" role="progressbar" style="width: 20%"></div>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- Recommendations Section -->
                    <div class="row">
                        <div class="col-md-12">
                            <div class="card">
                                <div class="dashboard-card-header">
                                    Data-Driven Recommendations
                                </div>
                                <div class="card-body">
                                    <div class="row">
                                        <div class="col-md-4">
                                            <div class="card bg-light mb-3">
                                                <div class="card-body">
                                                    <h5 class="card-title"><i class="fas fa-traffic-light text-success me-2"></i> Enforcement</h5>
                                                    <ul>
                                                        <li>Increase patrols on NH48 (Delhi-Jaipur section)</li>
                                                        <li>Target speed checks 6-10 PM</li>
                                                        <li>Weekend DUI checkpoints near pubs</li>
                                                    </ul>
                                                </div>
                                            </div>
                                        </div>
                                        <div class="col-md-4">
                                            <div class="card bg-light mb-3">
                                                <div class="card-body">
                                                    <h5 class="card-title"><i class="fas fa-road text-primary me-2"></i> Infrastructure</h5>
                                                    <ul>
                                                        <li>Install rumble strips at 5 high-fatality curves</li>
                                                        <li>Improve lighting on NH44 (priority sections)</li>
                                                        <li>Separate two-wheeler lanes in urban areas</li>
                                                    </ul>
                                                </div>
                                            </div>
                                        </div>
                                        <div class="col-md-4">
                                            <div class="card bg-light mb-3">
                                                <div class="card-body">
                                                    <h5 class="card-title"><i class="fas fa-bullhorn text-warning me-2"></i> Awareness</h5>
                                                    <ul>
                                                        <li>Diwali safety campaign targeting 18-35 age group</li>
                                                        <li>Social media blitz weekend before holidays</li>
                                                        <li>Partner with food delivery apps for rider alerts</li>
                                                    </ul>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Footer -->
    <footer class="bg-dark text-white py-3">
        <div class="container-fluid">
            <div class="row">
                <div class="col-md-6">
                    <p class="mb-0">© 2023 Traffic Safety Analytics | Open Source Project on GitHub</p>
                </div>
                <div class="col-md-6 text-md-end">
                    <p class="mb-0">GitHub: <a href="https://github.com/yourusername/traffic-safety-india" class="text-white">traffic-safety-india</a> | Last Updated: <span id="lastUpdated">Loading...</span></p>
                </div>
            </div>
        </div>
    </footer>

    <!-- JavaScript Libraries -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chartjs-plugin-datalabels/2.0.0/chartjs-plugin-datalabels.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/leaflet.heat@0.2.0/dist/leaflet-heat.js"></script>
    
    <script>
        // Load all data
        async function loadData() {
            try {
                // Update last updated time
                const now = new Date();
                document.getElementById('lastUpdated').textContent = now.toLocaleString();
                
                // Refresh real-time data
                realtimeIncidents = await fetchRealTimeData();
                updateRealTimeTable();
                
                // Refresh charts
                updateAllCharts();
            } catch (error) {
                console.error("Error loading data:", error);
            }
        }

        // Load all data
        async function loadData() {
            try {
                // Update last updated time
                const now = new Date();
                document.getElementById('lastUpdated').textContent = now.toLocaleString();
                
                // Refresh real-time data
                realtimeIncidents = await fetchRealTimeData();
                updateRealTimeTable();
                
                // Refresh charts
                updateAllCharts();
            } catch (error) {
                console.error("Error loading data:", error);
            }
        }

        // Initialize last updated time
        document.getElementById('lastUpdated').textContent = new Date().toLocaleString();

        // Sample accident data (in real implementation, this would come from an API)
        const accidentData = {
            yearly: 153972,
            trend: 5.2,
            causes: [
                { name: 'Overspeeding', percentage: 48, risk: 'high' },
                { name: 'Drunken Driving', percentage: 23, risk: 'high' },
                { name: 'Vehicle Defects', percentage: 12, risk: 'medium' },
                { name: 'Road Conditions', percentage: 17, risk: 'medium' }
            ],
            hotspots: [
                { name: 'NH44 (Kanyakumari to Srinagar)', deaths: 8742 },
                { name: 'NH48 (Delhi to Chennai)', deaths: 6512 },
                { name: 'NH27 (Porbandar to Silchar)', deaths: 5223 }
            ],
            vehicles: [
                { type: 'Two-wheelers', percentage: 37 },
                { type: 'Cars', percentage: 22 },
                { type: 'Trucks', percentage: 18 },
                { type: 'Buses', percentage: 11 },
                { type: 'Other', percentage: 12 }
            ],
            timePatterns: [
                { hour: '12-3 AM', accidents: 8, fatalities: 35 },
                { hour: '3-6 AM', accidents: 5, fatalities: 22 },
                { hour: '6-9 AM', accidents: 12, fatalities: 18 },
                { hour: '9AM-12PM', accidents: 9, fatalities: 15 },
                { hour: '12-3 PM', accidents: 10, fatalities: 17 },
                { hour: '3-6 PM', accidents: 14, fatalities: 20 },
                { hour: '6-9 PM', accidents: 23, fatalities: 38 },
                { hour: '9-12 PM', accidents: 18, fatalities: 32 }
            ],
            demographics: {
                age: [
                    { range: '15-24', percentage: 32 },
                    { range: '25-34', percentage: 28 },
                    { range: '35-44', percentage: 18 },
                    { range: '45-54', percentage: 12 },
                    { range: '55+', percentage: 10 }
                ],
                gender: [
                    { gender: 'Male', percentage: 82 },
                    { gender: 'Female', percentage: 18 }
                ]
            }
        };

        // Real-time incident data
        const realtimeIncidents = [
            { 
                time: '10:45 AM', 
                location: 'NH48, Km 45 (Delhi-Gurgaon)', 
                type: 'Multi-vehicle collision', 
                severity: 'High', 
                cause: 'Overspeeding',
                predicted: true
            },
            { 
                time: '9:30 AM', 
                location: 'MG Road, Bangalore', 
                type: 'Pedestrian hit', 
                severity: 'Medium', 
                cause: 'Signal jump',
                predicted: false
            },
            { 
                time: '8:15 AM', 
                location: 'Eastern Express Highway, Mumbai', 
                type: 'Lane change collision', 
                severity: 'Low', 
                cause: 'Distracted driving',
                predicted: true
            },
            { 
                time: '7:50 AM', 
                location: 'Anna Salai, Chennai', 
                type: 'Two-wheeler skid', 
                severity: 'Medium', 
                cause: 'Wet road',
                predicted: false
            }
        ];

        // Initialize all charts
        document.addEventListener('DOMContentLoaded', async function() {
            // Initial load
            await loadData();
            
            // Auto-refresh every 5 minutes
            setInterval(async () => {
                await loadData();
                console.log('Data refreshed');
            }, 300000);
            // Gauge Chart for Prediction Accuracy
            const gaugeCtx = document.getElementById('accuracyGauge').getContext('2d');
            new Chart(gaugeCtx, {
                type: 'doughnut',
                data: {
                    datasets: [{
                        data: [83.7, 16.3],
                        backgroundColor: ['#28a745', '#f8f9fa'],
                        borderWidth: 0
                    }]
                },
                options: {
                    rotation: -90,
                    circumference: 180,
                    plugins: {
                        legend: { display: false },
                        tooltip: { enabled: false }
                    },
                    cutout: '80%'
                }
            });

            // Time Pattern Chart
            const timeCtx = document.getElementById('timeChart').getContext('2d');
            new Chart(timeCtx, {
                type: 'bar',
                data: {
                    labels: accidentData.timePatterns.map(item => item.hour),
                    datasets: [
                        {
                            label: 'Accidents',
                            data: accidentData.timePatterns.map(item => item.accidents),
                            backgroundColor: 'rgba(54, 162, 235, 0.7)',
                            borderColor: 'rgba(54, 162, 235, 1)',
                            borderWidth: 1
                        },
                        {
                            label: 'Fatalities (%)',
                            data: accidentData.timePatterns.map(item => item.fatalities),
                            backgroundColor: 'rgba(255, 99, 132, 0.7)',
                            borderColor: 'rgba(255, 99, 132, 1)',
                            borderWidth: 1,
                            type: 'line',
                            tension: 0.1
                        }
                    ]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'top',
                        },
                        title: {
                            display: false
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            });

            // Vehicle Type Chart
            const vehicleCtx = document.getElementById('vehicleChart').getContext('2d');
            new Chart(vehicleCtx, {
                type: 'pie',
                data: {
                    labels: accidentData.vehicles.map(item => item.type),
                    datasets: [{
                        data: accidentData.vehicles.map(item => item.percentage),
                        backgroundColor: [
                            'rgba(255, 99, 132, 0.7)',
                            'rgba(54, 162, 235, 0.7)',
                            'rgba(255, 206, 86, 0.7)',
                            'rgba(75, 192, 192, 0.7)',
                            'rgba(153, 102, 255, 0.7)'
                        ],
                        borderColor: [
                            'rgba(255, 99, 132, 1)',
                            'rgba(54, 162, 235, 1)',
                            'rgba(255, 206, 86, 1)',
                            'rgba(75, 192, 192, 1)',
                            'rgba(153, 102, 255, 1)'
                        ],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'right',
                        },
                        title: {
                            display: false
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    return `${context.label}: ${context.raw}% of deaths`;
                                }
                            }
                        }
                    }
                }
            });

            // Age Distribution Chart
            const ageCtx = document.getElementById('ageChart').getContext('2d');
            new Chart(ageCtx, {
                type: 'bar',
                data: {
                    labels: accidentData.demographics.age.map(item => item.range),
                    datasets: [{
                        label: 'Percentage of Victims',
                        data: accidentData.demographics.age.map(item => item.percentage),
                        backgroundColor: 'rgba(54, 162, 235, 0.7)',
                        borderColor: 'rgba(54, 162, 235, 1)',
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: false
                        },
                        title: {
                            display: false
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: 'Percentage (%)'
                            }
                        }
                    }
                }
            });

            // Gender Distribution Chart
            const genderCtx = document.getElementById('genderChart').getContext('2d');
            new Chart(genderCtx, {
                type: 'doughnut',
                data: {
                    labels: accidentData.demographics.gender.map(item => item.gender),
                    datasets: [{
                        data: accidentData.demographics.gender.map(item => item.percentage),
                        backgroundColor: [
                            'rgba(54, 162, 235, 0.7)',
                            'rgba(255, 99, 132, 0.7)'
                        ],
                        borderColor: [
                            'rgba(54, 162, 235, 1)',
                            'rgba(255, 99, 132, 1)'
                        ],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'right',
                        },
                        title: {
                            display: false
                        }
                    }
                }
            });

            // Prediction Chart
            const predictionCtx = document.getElementById('predictionChart').getContext('2d');
            new Chart(predictionCtx, {
                type: 'bar',
                data: {
                    labels: ['Normal Weekend', 'Upcoming Diwali'],
                    datasets: [{
                        label: 'Expected Accidents',
                        data: [100, 120],
                        backgroundColor: [
                            'rgba(54, 162, 235, 0.7)',
                            'rgba(255, 159, 64, 0.7)'
                        ],
                        borderColor: [
                            'rgba(54, 162, 235, 1)',
                            'rgba(255, 159, 64, 1)'
                        ],
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: false
                        },
                        title: {
                            display: false
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            title: {
                                display: true,
                                text: 'Relative Accident Risk'
                            }
                        }
                    }
                }
            });

            // Update real-time table function
            function updateRealTimeTable() {
                const realtimeTable = document.getElementById('realtimeTable');
                realtimeTable.innerHTML = ''; // Clear existing rows
            realtimeIncidents.forEach(incident => {
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${incident.time}</td>
                    <td>${incident.location}</td>
                    <td>${incident.type}</td>
                    <td>
                        <span class="badge ${incident.severity === 'High' ? 'bg-danger' : incident.severity === 'Medium' ? 'bg-warning' : 'bg-success'}">
                            ${incident.severity}
                        </span>
                    </td>
                    <td>${incident.cause}</td>
                    <td>
                        <button class="btn btn-sm ${incident.predicted ? 'btn-info' : 'btn-secondary'}">
                            ${incident.predicted ? 'Predicted' : 'New'}
                        </button>
                    </td>
                `;
                realtimeTable.appendChild(row);
            });

            // Initialize Map
            initializeMap();
        });

        // Update all charts
        function updateAllCharts() {
            updateTimeChart();
            updateVehicleChart();
            updateAgeChart();
            updateGenderChart();
            updatePredictionChart();
        }

        // Update all charts
        function updateAllCharts() {
            updateTimeChart();
            updateVehicleChart();
            updateAgeChart();
            updateGenderChart();
            updatePredictionChart();
        }

        // Initialize Leaflet Map with Heatmap
        function initializeMap() {
            const map = L.map('map').setView([20.5937, 78.9629], 5);

            // Add base tile layer
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>'
            }).addTo(map);

            // Fetch real heatmap data from API
            const heatResponse = await fetch('https://your-api-endpoint.com/heatmap');
            const heatPoints = await heatResponse.json() || [
                [28.7041, 77.1025, 0.8],  // Delhi
                [19.0760, 72.8777, 0.7],  // Mumbai
                [13.0827, 80.2707, 0.6],  // Chennai
                [12.9716, 77.5946, 0.5],  // Bangalore
                [22.5726, 88.3639, 0.45], // Kolkata
                [17.3850, 78.4867, 0.4],  // Hyderabad
                [26.9124, 75.7873, 0.35], // Jaipur
                [18.5204, 73.8567, 0.3],  // Pune
                [23.0225, 72.5714, 0.25], // Ahmedabad
                [25.5941, 85.1376, 0.2]   // Patna
            ];

            // Create heat layer
            const heatLayer = L.heatLayer(heatPoints, {
                radius: 25,
                blur: 15,
                maxZoom: 17,
                max: 1.0,
                gradient: {0.4: 'blue', 0.6: 'lime', 0.8: 'yellow', 1.0: 'red'}
            }).addTo(map);

            // Add some markers for hotspots
            const hotspots = [
                {
                    coords: [28.7041, 77.1025],
                    title: "Delhi",
                    description: "National Capital Region - High accident density (12 deaths/day average)"
                },
                {
                    coords: [19.0760, 72.8777],
                    title: "Mumbai",
                    description: "Western Coastal City - Heavy traffic congestion areas"
                },
                {
                    coords: [13.0827, 80.2707],
                    title: "Chennai",
                    description: "Southern Metro - High two-wheeler accident zone"
                }
            ];

            hotspots.forEach(hotspot => {
                L.marker(hotspot.coords)
                    .addTo(map)
                    .bindPopup(`<strong>${hotspot.title}</strong><br>${hotspot.description}`)
                    .openPopup();
            });

            // Layer control
            const baseLayers = {
                "Standard Map": L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png')
            };

            const overlays = {
                "Heatmap": heatLayer
            };

            L.control.layers(baseLayers, overlays).addTo(map);
        }
    </script>
</body>
</html>
