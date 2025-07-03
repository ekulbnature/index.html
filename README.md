<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ontario's 30x30 Progress</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
        }
        .chart-container {
            position: relative;
            width: 400px;
            height: 400px;
        }
        .donut-chart {
            width: 100%;
            height: 100%;
        }
        .legend {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            margin-top: 20px;
            gap: 10px;
        }
        .legend-item {
            display: flex;
            align-items: center;
            margin-right: 15px;
        }
        .legend-color {
            width: 15px;
            height: 15px;
            margin-right: 5px;
            border-radius: 3px;
        }
        .title {
            font-size: 1.5em;
            font-weight: bold;
            margin-bottom: 10px;
        }
        .description {
            max-width: 500px;
            text-align: center;
            margin-bottom: 20px;
            line-height: 1.5;
        }
        .percentage-display {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            font-size: 24px;
            font-weight: bold;
        }
        .goal-line {
            stroke-dasharray: 5,5;
            stroke: #9e9e9e;
            stroke-width: 2;
        }
        .goal-text {
            font-size: 12px;
            fill: #666;
        }
    </style>
</head>
<body>
    <div class="title">Ontario's Progress Toward 30x30 Goal</div>
    <div class="description">
        This chart shows progress toward protecting 30% of Ontario's land by 2030.
        Currently 11% is protected, with an additional 7% identified as potential protected areas.
    </div>
    
    <div class="chart-container">
        <div class="donut-chart" id="donutChart"></div>
        <div class="percentage-display" id="percentageDisplay"></div>
    </div>
    
    <div class="legend" id="legend"></div>

    <script>
        // Data - now scaled to 30% being 100% of the chart
        const currentProtected = (11/30)*100;  // 36.67% of goal
        const potentialAdditional = (7/30)*100; // 23.33% of goal
        const remaining = 100 - currentProtected - potentialAdditional; // 40% of goal
        
        const data = [
            {label: "Remaining to Goal", value: remaining, color: "#e0e0e0"},
            {label: "Potential Additional Protected Areas", value: potentialAdditional, color: "#ffa726"},
            {label: "Currently Protected Areas", value: currentProtected, color: "#4caf50"},
        ];
        
        const width = 400;
        const height = 400;
        const radius = Math.min(width, height) / 2;
        const donutWidth = 60;
        
        // Create SVG
        const svg = d3.select("#donutChart")
            .append("svg")
            .attr("width", width)
            .attr("height", height)
            .append("g")
            .attr("transform", `translate(${width/2}, ${height/2})`);
        
        // Create arc generator
        const arc = d3.arc()
            .innerRadius(radius - donutWidth)
            .outerRadius(radius);
        
        // Create pie generator
        const pie = d3.pie()
            .value(d => d.value)
            .sort(null);
        
        // Draw the goal outline (100% = 30% of Ontario's land)
        const goalOutline = svg.append("circle")
            .attr("r", radius)
            .attr("fill", "none")
            .attr("stroke", "#9e9e9e")
            .attr("stroke-width", 2)
            .attr("stroke-dasharray", "5,5")
            .attr("opacity", 0.7);
        
        // Add goal text
        svg.append("text")
            .attr("class", "goal-text")
            .attr("x", 0)
            .attr("y", -radius - 10)
            .attr("text-anchor", "middle")
            .text("30% of Ontario's Land");
        
        // Create the main donut chart
        const paths = svg.selectAll("path")
            .data(pie(data))
            .enter()
            .append("path")
            .attr("d", arc)
            .attr("fill", d => d.data.color)
            .attr("opacity", 0)
            .attr("stroke", "#fff")
            .attr("stroke-width", 1);
        
        // Add legend
        const legend = d3.select("#legend");
        
        data.forEach(d => {
            const legendItem = legend.append("div")
                .attr("class", "legend-item");
            
            legendItem.append("div")
                .attr("class", "legend-color")
                .style("background-color", d.color);
            
            // Convert back to actual percentages of Ontario's land
            const actualPercent = (d.value/100)*30;
            legendItem.append("div")
                .text(`${d.label} (${actualPercent.toFixed(1)}% of Ontario)`);
        });
        
        // Add goal to legend
        const goalLegend = legend.append("div")
            .attr("class", "legend-item");
        
        goalLegend.append("div")
            .attr("class", "legend-color")
            .style("background-color", "none")
            .style("border", "2px dashed #9e9e9e");
        
        goalLegend.append("div")
            .text("30x30 Goal (30% of Ontario)");
        
        // Animation sequence
        function animateChart() {
            // Reset display
            d3.select("#percentageDisplay").text("");
            
            // Show goal outline first
            goalOutline.transition()
                .duration(500)
                .attr("opacity", 1)
                .on("end", function() {
                    // Then show current protected areas (green)
                    svg.selectAll("path")
                        .filter((d, i) => i === 2) // Currently protected is last in array
                        .transition()
                        .duration(1000)
                        .attr("opacity", 1)
                        .on("end", function() {
                            const currentPercent = (currentProtected/100)*30;
                            d3.select("#percentageDisplay").text(`${currentPercent.toFixed(1)}% Protected`);
                            
                            // Then show potential areas (orange)
                            svg.selectAll("path")
                                .filter((d, i) => i === 1)
                                .transition()
                                .duration(1000)
                                .attr("opacity", 1)
                                .on("end", function() {
                                    const totalWithPotential = ((currentProtected + potentialAdditional)/100)*30;
                                    d3.select("#percentageDisplay").text(`${totalWithPotential.toFixed(1)}% Protected + Potential`);
                                    
                                    // Finally show remaining (grey)
                                    svg.selectAll("path")
                                        .filter((d, i) => i === 0)
                                        .transition()
                                        .duration(1000)
                                        .attr("opacity", 1)
                                        .on("end", function() {
                                            d3.select("#percentageDisplay").html(`30% Goal<br><span style="font-size:16px">(100% of target)</span>`);
                                        });
                                });
                        });
                });
        }
        
        // Start animation automatically after short delay
        setTimeout(animateChart, 500);
    </script>
</body>
</html>
