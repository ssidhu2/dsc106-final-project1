<script>
    import * as d3 from 'd3'; 
    import { onMount } from 'svelte';
    import { scaleLinear } from 'd3-scale';
    import { select } from 'd3-selection';

    export let data;
    let targetYear = 2015;
    let filteredData = [];

    onMount(() => {
        // Filter data based on the target year and country code
        filteredData = data.filter(d => d.year === targetYear);
        
        // Extract Gini index (SI.POV.GINI) and GDP (current US$) (NY.GDP.MKTP.CD) values from filtered data
        const giniValues = filteredData.filter(d => d.indicator_code === "SI.POV.GINI").map(d => d.value);
        const gdpValues = filteredData.filter(d => d.indicator_code === "NY.GDP.MKTP.CD").map(d => d.value);
        const countryName = filteredData.filter(d => d.indicator_code === "SI.POV.GINI").map(d => d.country_name);
        
        // Filtered arrays
        const filteredGiniValues = [];
        const filteredGdpValues = [];
        const filteredCountryName = [];

        // Iterate through values and filter out zeros
        for (let i = 0; i < giniValues.length; i++) {
            if (giniValues[i] !== 0 && gdpValues[i] !== 0) {
                filteredGiniValues.push(giniValues[i]);
                filteredGdpValues.push(gdpValues[i]);
                filteredCountryName.push(countryName[i]);
            }
        }

        // Set up the SVG dimensions
        const margin = { top: 20, right: 50, bottom: 50, left: 50 };
        const width = 1000 - margin.left - margin.right;
        const height = 600 - margin.top - margin.bottom;

        // Append SVG container to the body
        const svg = select("#scatterplot")
            .attr("width", width + margin.left + margin.right)
            .attr("height", height + margin.top + margin.bottom)
            .append("g")
            .attr("transform", `translate(${margin.left},${margin.top})`);

        // Define scales for x and y axes
        const xScale = scaleLinear()
            .domain([Math.min(...filteredGiniValues), Math.max(...filteredGiniValues)])
            .range([0, width]);

        const yScale = scaleLinear()
            .domain([Math.min(...filteredGdpValues), Math.max(...filteredGdpValues)])
            .range([height, 0]);

        // Plot points with tooltip
        svg.selectAll(".dot")
            .data(filteredCountryName)
            .enter().append("circle")
            .attr("class", "dot")
            .attr("cx", (d, i) => xScale(filteredGiniValues[i]))
            .attr("cy", (d, i) => yScale(filteredGdpValues[i]))
            .attr("r", 5) // radius of the circles
            .append("title")
            .text((d, i) => `Country: ${d}\nGDP: ${(filteredGdpValues[i] / 1e9).toFixed(3)}B\nGini: ${filteredGiniValues[i]}`);

        // Add x-axis
        svg.append("g")
            .attr("transform", `translate(0,${height})`)
            .call(d3.axisBottom(xScale));

        // Add y-axis
        svg.append("g")
            .call(d3.axisLeft(yScale));

        // Add axis labels
        svg.append("text")
            .attr("text-anchor", "middle")
            .attr("transform", `translate(${width / 2},${height + margin.top + 10})`)
            .text("Gini Index");

        svg.append("text")
            .attr("text-anchor", "middle")
            .attr("transform", `translate(${-margin.left + 10},${height / 2})rotate(-90)`)
            .text("GDP (current US$)");
    });
</script>

<svg id="scatterplot" width="800" height="600"></svg>
