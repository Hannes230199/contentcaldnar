<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RSMG Interactive Content Calendar | August 2025</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chosen Palette: Warm Neutral Harmony -->
    <!-- Application Structure Plan: The SPA is designed as an interactive dashboard. The top-level consists of a filter control panel for immediate user interaction, followed by a data visualization section (a dynamic chart and key stats) for a high-level overview. The main content area is a responsive grid of event cards. This structure was chosen to transform the static table into a functional tool. It allows users to fluidly move from a broad overview (the chart) to specific, filtered information (the event cards) and finally to granular details (expanding a card). This flow directly supports a content planner's workflow: see the big picture, find relevant events, then access the specific strategy. -->
    <!-- Visualization & Content Choices: The calendar data from the report is transformed into several interactive components. Goal: Organize/Filter -> Presentation: A filter bar with dropdowns and buttons -> Interaction: Users select a market, significance, or sport to instantly update the view -> Justification: This is the core function, allowing users to personalize the data. Goal: Compare -> Presentation: A dynamic Doughnut Chart (Chart.js/Canvas) showing the distribution of events by sport -> Interaction: The chart redraws with every filter change, providing immediate visual feedback on the composition of the selected events -> Justification: It provides a quick, visual summary that's easier to digest than a list. Goal: Inform/Explore -> Presentation: Event Cards in a responsive grid -> Interaction: Clicking a card expands to show detailed content angles -> Justification: This keeps the main interface clean while providing deep information on demand. -->
    <!-- CONFIRMATION: NO SVG graphics used. NO Mermaid JS used. -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc; /* slate-50 */
        }
        .chart-container {
            position: relative;
            width: 100%;
            max-width: 400px;
            margin-left: auto;
            margin-right: auto;
            height: 350px;
            max-height: 400px;
        }
        @media (min-width: 768px) {
            .chart-container {
                height: 400px;
            }
        }
        /* Custom transition for smooth card expansion */
        details .content {
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.5s ease-in-out;
        }
        details[open] .content {
            max-height: 1000px; /* Large enough to not clip content */
        }
    </style>
</head>
<body class="text-slate-800">

    <div class="container mx-auto p-4 md:p-8">

        <header class="text-center mb-8">
            <h1 class="text-3xl md:text-4xl font-bold text-slate-900">RSMG Interactive Content Calendar</h1>
            <p class="text-lg text-slate-600 mt-2">August 2025</p>
        </header>

        <section id="filters" class="p-4 bg-white rounded-xl shadow-sm mb-8 sticky top-4 z-10">
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
                <div>
                    <label for="market-filter" class="block text-sm font-medium text-slate-700 mb-1">Filter by Market</label>
                    <select id="market-filter" class="w-full p-2 border border-slate-300 rounded-md shadow-sm focus:ring-2 focus:ring-cyan-500 focus:border-cyan-500 transition">
                        <option value="ALL">All Markets</option>
                        <option value="RO">Romania (RO)</option>
                        <option value="BG">Bulgaria (BG)</option>
                        <option value="HU">Hungary (HU)</option>
                        <option value="SK">Slovakia (SK)</option>
                        <option value="RS">Serbia (RS)</option>
                        <option value="PL">Poland (PL)</option>
                        <option value="PT">Portugal (PT)</option>
                        <option value="CH">Switzerland (CH)</option>
                    </select>
                </div>
                <div>
                    <label for="sport-filter" class="block text-sm font-medium text-slate-700 mb-1">Filter by Sport</label>
                    <select id="sport-filter" class="w-full p-2 border border-slate-300 rounded-md shadow-sm focus:ring-2 focus:ring-cyan-500 focus:border-cyan-500 transition">
                        <option value="ALL">All Sports</option>
                    </select>
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Filter by Significance</label>
                    <div id="significance-filter" class="flex space-x-2">
                        <button data-sig="ALL" class="flex-1 p-2 border rounded-md transition text-white bg-cyan-600">All</button>
                        <button data-sig="High" class="flex-1 p-2 border rounded-md transition bg-white hover:bg-slate-100">High</button>
                        <button data-sig="Medium" class="flex-1 p-2 border rounded-md transition bg-white hover:bg-slate-100">Medium</button>
                    </div>
                </div>
            </div>
        </section>

        <section id="dashboard" class="mb-8 grid grid-cols-1 lg:grid-cols-3 gap-8 items-center">
            <div class="lg:col-span-1 bg-white p-6 rounded-xl shadow-sm">
                 <h2 class="text-xl font-bold text-center text-slate-900 mb-4">Event Distribution by Sport</h2>
                 <p class="text-center text-sm text-slate-500 mb-4">This chart dynamically updates based on your filters to show the mix of sports for the selected criteria.</p>
                <div class="chart-container">
                    <canvas id="sports-chart"></canvas>
                </div>
            </div>
            <div class="lg:col-span-2 grid grid-cols-1 sm:grid-cols-2 gap-6">
                <div class="p-6 bg-white rounded-xl shadow-sm">
                    <h3 class="font-bold text-slate-900 text-lg">Filtered Events</h3>
                    <p id="total-events" class="text-4xl font-extrabold text-cyan-600 mt-2">0</p>
                    <p class="text-slate-500 mt-1">Total events matching your criteria.</p>
                </div>
                 <div class="p-6 bg-white rounded-xl shadow-sm">
                    <h3 class="font-bold text-slate-900 text-lg">High-Significance Events</h3>
                    <p id="high-sig-events" class="text-4xl font-extrabold text-cyan-600 mt-2">0</p>
                    <p class="text-slate-500 mt-1">Major global and market-specific events.</p>
                </div>
                 <div class="p-6 bg-white rounded-xl shadow-sm col-span-1 sm:col-span-2">
                    <h3 class="font-bold text-slate-900 text-lg">About this Calendar</h3>
                    <p class="text-slate-600 mt-2 text-sm">This interactive tool translates our static content calendar into a dynamic planning resource. Use the filters above to isolate events by market, sport, or importance. Click on any event card's 'View Details' to see tailored content angles and strategic opportunities, helping us deliver the best experience to sports fans everywhere.</p>
                </div>
            </div>
        </section>
        
        <main id="event-grid" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-6">
            
        </main>
    </div>

    <script>
        const eventData = [
            { date: "Aug 8-10", name: "Premier League Opening Weekend", sport: "Football", location: "England", significance: "High", relevance: ["ALL", "PT", "PL"], angles: "Pre: Season predictions, fantasy league tips. Live: Live blogs, goal clips on social. Post: Match analysis, 'Team of the Week' featuring local heroes.", formats: ["Article", "Video", "Social", "Podcast"] },
            { date: "Aug 12", name: "UEFA Super Cup", sport: "Football", location: "TBD", significance: "High", relevance: ["ALL"], angles: "Deep-dive tactical preview. Historical look back at the fixture. Player-to-watch profiles. Post-match '5 Things We Learned.'", formats: ["Video", "Article", "Interactive"] },
            { date: "Mid-Aug", name: "Start of Domestic Leagues", sport: "Football", location: "Europe", significance: "Medium", relevance: ["RO", "BG", "HU", "SK", "RS", "PL", "PT", "CH"], angles: "Team-by-team season guides. Fan expectation interviews. 'Derby Day' countdowns. Transfer window analysis for each league.", formats: ["Article", "Photo Gallery", "Social Media Polls"] },
            { date: "Aug 19 - Sep 7", name: "La Vuelta a España", sport: "Cycling", location: "Spain", significance: "Medium", relevance: ["PT", "CH"], angles: "Stage-by-stage analysis. 'Hero of the Day' profiles. Interactive map of the route. Highlight the grueling nature of a Grand Tour.", formats: ["Interactive", "Video", "Article"] },
            { date: "Aug 23", name: "Diamond League: Silesia", sport: "Athletics", location: "Chorzów, Poland", significance: "Medium", relevance: ["PL"], angles: "Host Focus: Venue guides, interviews with Polish athletes. Global: 'Clash of the Titans' narrative for big head-to-head races. Explainer on what it takes to win.", formats: ["Video", "Social", "Article"] },
            { date: "Aug 25 - Sep 7", name: "US Open", sport: "Tennis", location: "New York, USA", significance: "High", relevance: ["RS", "PL", "CH"], angles: "'Path to the Final' bracket challenge. Daily wrap-up show/podcast. Focus on human-interest stories of underdog players. Fashion and celebrity-spotting at the event.", formats: ["Podcast", "Video", "Article", "Social"] },
            { date: "Aug 29-31", name: "Formula 1 - Dutch Grand Prix", sport: "Motorsport", location: "Zandvoort, NL", significance: "High", relevance: ["ALL"], angles: "Technical analysis of the track. Fan zone experience features. Data-driven race predictions. Live race commentary and driver radio highlights.", formats: ["Data Viz", "Video", "Live Blog"] },
            { date: "TBD", name: "FIBA World Cup Qualifiers", sport: "Basketball", location: "Various", significance: "Medium", relevance: ["RS", "PT", "GR"], angles: "National team roster breakdowns. 'Key to Victory' analysis for crucial games. Post-game player interviews and press conference summaries.", formats: ["Article", "Social", "Video"] }
        ];

        const state = {
            market: 'ALL',
            sport: 'ALL',
            significance: 'ALL'
        };

        const eventGrid = document.getElementById('event-grid');
        const marketFilter = document.getElementById('market-filter');
        const sportFilter = document.getElementById('sport-filter');
        const significanceFilter = document.getElementById('significance-filter');
        const totalEventsEl = document.getElementById('total-events');
        const highSigEventsEl = document.getElementById('high-sig-events');
        
        let sportsChart;

        function getSignificanceClass(sig) {
            switch (sig) {
                case 'High': return 'bg-red-100 text-red-800';
                case 'Medium': return 'bg-amber-100 text-amber-800';
                default: return 'bg-slate-100 text-slate-800';
            }
        }
        
        function createSportIcon(sport) {
            const icons = {
                'Football': '⚽️', 'Cycling': '🚴', 'Athletics': '🏃',
                'Tennis': '🎾', 'Motorsport': '🏎️', 'Basketball': '🏀'
            };
            return icons[sport] || '🏅';
        }

        function renderEvents() {
            const filteredEvents = eventData.filter(event => {
                const marketMatch = state.market === 'ALL' || event.relevance.includes(state.market) || event.relevance.includes('ALL');
                const sportMatch = state.sport === 'ALL' || event.sport === state.sport;
                const significanceMatch = state.significance === 'ALL' || event.significance === state.significance;
                return marketMatch && sportMatch && significanceMatch;
            });

            eventGrid.innerHTML = '';
            if (filteredEvents.length === 0) {
                eventGrid.innerHTML = `<p class="md:col-span-2 xl:col-span-3 text-center text-slate-500 py-10">No events match the current filters.</p>`;
            } else {
                filteredEvents.forEach(event => {
                    const card = document.createElement('div');
                    card.className = 'bg-white rounded-xl shadow-sm transition hover:shadow-lg';
                    card.innerHTML = `
                        <div class="p-5">
                            <div class="flex justify-between items-start">
                                <p class="text-xs text-slate-500">${event.date}</p>
                                <span class="${getSignificanceClass(event.significance)} text-xs font-semibold mr-2 px-2.5 py-0.5 rounded-full">${event.significance}</span>
                            </div>
                            <h3 class="font-bold text-lg mt-2 text-slate-900">${createSportIcon(event.sport)} ${event.name}</h3>
                            <p class="text-sm text-slate-600 mt-1">📍 ${event.location}</p>
                            <div class="mt-3 text-xs">
                                <span class="font-semibold">Markets:</span>
                                ${event.relevance.map(m => `<span class="inline-block bg-slate-100 rounded-full px-2 py-1 font-medium text-slate-700 mr-1 mb-1">${m}</span>`).join('')}
                            </div>
                        </div>
                        <details class="text-sm">
                            <summary class="px-5 py-3 bg-slate-50 cursor-pointer font-medium text-cyan-700 hover:bg-slate-100 transition rounded-b-xl list-none flex justify-between items-center">
                                View Details
                                <span class="transform transition-transform duration-300 details-arrow">▼</span>
                            </summary>
                            <div class="content">
                                <div class="p-5 border-t border-slate-200">
                                    <h4 class="font-semibold mb-2">Content Angles & Opportunities</h4>
                                    <p class="text-slate-600 mb-4">${event.angles}</p>
                                    <h4 class="font-semibold mb-2">Suggested Formats</h4>
                                    <div class="flex flex-wrap gap-2">
                                        ${event.formats.map(f => `<span class="bg-cyan-100 text-cyan-800 px-2 py-1 rounded-md text-xs font-medium">${f}</span>`).join('')}
                                    </div>
                                </div>
                            </div>
                        </details>
                    `;
                    eventGrid.appendChild(card);
                });
            }

            totalEventsEl.textContent = filteredEvents.length;
            highSigEventsEl.textContent = filteredEvents.filter(e => e.significance === 'High').length;
            updateChart(filteredEvents);
        }
        
        function populateSportFilter() {
            const sports = [...new Set(eventData.map(e => e.sport))];
            sports.sort();
            sports.forEach(sport => {
                const option = document.createElement('option');
                option.value = sport;
                option.textContent = sport;
                sportFilter.appendChild(option);
            });
        }
        
        function setupChart() {
            const ctx = document.getElementById('sports-chart').getContext('2d');
            sportsChart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: [],
                    datasets: [{
                        label: 'Events by Sport',
                        data: [],
                        backgroundColor: [
                            '#06b6d4', '#67e8f9', '#a5f3fc', '#cffafe',
                            '#0891b2', '#22d3ee', '#67e8f9', '#a5f3fc'
                        ],
                        borderColor: '#f8fafc',
                        borderWidth: 4
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: {
                            position: 'bottom',
                            labels: {
                                font: { size: 12 }
                            }
                        },
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed !== null) {
                                        label += context.parsed;
                                    }
                                    return label;
                                }
                            }
                        }
                    }
                }
            });
        }
        
        function updateChart(filteredEvents) {
            const sportCounts = filteredEvents.reduce((acc, event) => {
                acc[event.sport] = (acc[event.sport] || 0) + 1;
                return acc;
            }, {});

            const labels = Object.keys(sportCounts);
            const data = Object.values(sportCounts);

            sportsChart.data.labels = labels;
            sportsChart.data.datasets[0].data = data;
            sportsChart.update();
        }

        marketFilter.addEventListener('change', (e) => {
            state.market = e.target.value;
            renderEvents();
        });

        sportFilter.addEventListener('change', (e) => {
            state.sport = e.target.value;
            renderEvents();
        });

        significanceFilter.addEventListener('click', (e) => {
            if (e.target.tagName === 'BUTTON') {
                state.significance = e.target.dataset.sig;
                // Update active button style
                significanceFilter.querySelectorAll('button').forEach(btn => {
                    btn.classList.remove('bg-cyan-600', 'text-white');
                    btn.classList.add('bg-white', 'hover:bg-slate-100');
                });
                e.target.classList.add('bg-cyan-600', 'text-white');
                e.target.classList.remove('bg-white', 'hover:bg-slate-100');
                renderEvents();
            }
        });
        
        // Event delegation for details/summary toggle animation
        eventGrid.addEventListener('click', function(e) {
            const summary = e.target.closest('summary');
            if (summary) {
                const details = summary.parentElement;
                const arrow = summary.querySelector('.details-arrow');
                if (details.hasAttribute('open')) {
                    // It's about to close
                    e.preventDefault();
                    arrow.style.transform = 'rotate(0deg)';
                    const content = details.querySelector('.content');
                    content.style.maxHeight = '0';
                    setTimeout(() => { details.removeAttribute('open'); }, 500);
                } else {
                    // It's about to open
                    arrow.style.transform = 'rotate(180deg)';
                }
            }
        });

        document.addEventListener('DOMContentLoaded', () => {
            populateSportFilter();
            setupChart();
            renderEvents();
        });
    </script>
</body>
</html>
