## Creating a Sankey Diagram using d3 v4
(Based on [this webpage from bl.ocks.org](https://bl.ocks.org/d3noob/013054e8d7807dff76247b81b0e29030))
1. First you need to create a JSON file. As seen at the bottom of the webpage provided, there is a category of nodes and a category of links. The nodes are represented by the colorful bars shown at the top in the visualization. Links will be going in and out of the nodes. Each node has an ID number and a name, and each link has a source, target, and value. Take the link containing 4 widgets/units going from node3 to node4. Since node3 has an ID number of 3 and node4 has an ID number of 4, the link has a source of 3, a target of 4, and a value of 4. Note that there should be at most one link for every source-target pair of nodes.

2. If you want to get fancy and customize all of the colors of the nodes and links, you can look up CSS colors and add the attribute `“color”` to your nodes and links. For example, instead of just `{“node”:0, “name”: “node0”}`, you could make it `{“node”:0, “name”: “node0”, “color”: “#0092D0”}`. You will have to do this for each node and link in order for it to work.

3. Once your JSON file is in the desired format, copy the HTML code from the webpage. You’ll want to change your units according to what your links are representing, your margins according to how big you want the display area to be, potentially the color scheme, and the svg variable and how you’ll access it in the future (if your div id is “sankey”, you’ll want to put `d3.select(“#sankey”)`). Furthermore, you can also change the width and padding of the nodes, and make sure that you’re loading in the JSON file you created earlier from the right file on your computer. 

4. If you customized the colors of your nodes and/or links from step 2, then you’ll have to add a few lines. For customized link colors, in the declaration of the link variable, add a line that says 

   `.style(“stroke”, function(d){return d.color;})`. 

   For customized node colors, under the `node.append(“rect”)` section, add a line that says 

   `.style(“fill”, function(d) {return d.color;})`. 

5. Now when you load your page, you should be able to view the Sankey diagram!
