# Graph Visualization: `Visnet`

## 1. What is Visnet?

Visnet is a graph visualization library using WebGL to draw on HTML5 Canvas element. It outputs the desired visualization as a .html file and is compatible with Jupyter Notebook.

**Visnet In action:**

![Action](visnet.gif "Action")

Visnet is originally intended to be independent of any language as long as a compliant input is passed to `Generator`, the backbone program producing the canvas visualization. However, for the time being, Python is the language one can get some visualization work done quickly.


## 2. Installation
**Method 1:** Download the source code and append path of it to the Python PATH using the following instruction.

```python
import sys
sys.path.append("/path/to/folder/with/visnet")

import visnet as vn
```

## 3. Usage and Examples
Visnet consist of 3 types of visualizations at the moment. `Compound`, `Network`, `Heatmap`.

`Compound` is a visualization/container consisting of other *atomic visualizations*. Where as, `Network` and `Heatmap` are *atomic visuzalizations*.

### Network
`Network` is used to visualize a graph. It expects a `NetworkX.Graph` to be passed to construct a visualization. 

```python
import sys
sys.path.append("/path/to/folder/with/visnet") # introduce visnet package

import networkx as nx
import visnet as vn

nodes = [(0, {'contaminated': 0, 'class': 0, 'size': 4}),
         (1, {'contaminated': 1, 'class': 1, 'size': 7}),
         (2, {'contaminated': 0, 'class': 2, 'size': 5}),
         (3, {'contaminated': 0, 'class': 1, 'size': 5})]

links = [(0, 1, {'strength': 10}),
         (1, 2, {'strength': 12}),
         (2, 3, {'strength': 8}),
         (3, 0, {'strength': 11})]

G = nx.Graph()
G.add_nodes_from(nodes)
G.add_edges_from(links)
nw = vn.Network(G)
```

There are two functions to call to get an output

- ```nw.render_html()``` *A file named:* `visulization<object_ID>.html` *is produced.*
- ```nw.render_notebook()``` *Based on the previous one. Loads the html file to notebook.*

```python
nw.render_notebook()
```
Produces:
![Simple Network](https://imgur.com/JF9TsFX.png "Simple Network")


### Heatmap

`Heatmap` is used to visualize 0/1 matrices and it expects a Scipy Sparse Matrix or a Numpy Array to be passed to construct a visualization.


```python
import sys
sys.path.append("/path/to/folder/with/visnet") # introduce visnet package

import networkx as nx
import visnet as vn

row_labels = ['MN', 'CA', 'TX', 'NY']
col_labels = ['Clean', 'Rich', 'Crowded', 'Sunny', 'Global', 'Polluted', 'WeedLegal']
labels = {"row": row_labels, "col": col_labels}

matrix = np.random.randint(0, 2, [len(row_labels), len(col_labels)])
hm = vn.Heatmap(matrix, labels=labels)
hm.render_notebook()
```

Produces:
![Simple Heatmap](https://imgur.com/jADXHfH.png "Simple Heatmap")


### Compound
`Compound` is a container visualization and expects other *atomic visuzalizations* to be passed to construct a visualization. 

**Important Notice:** `Compound` visualization outputs a folder of child visualizations alongside the main html file associated with it. To display the visualization correctly, these must be run on a http server. An example:

```
/home/my_directory
   |-visualization1234.html
   |-visualization1234views
   |---visualization1235.html
   |---visualization1236.html
```

A `Compound` visualization (*visualization1234.html*) consisting of two other visualizations: *visualization1235.html* and *visualization1236.html*

`cd` to `/home/my_directory` and run the server:

```bash
python -m http.server 9000
```

Render the page on Chrome Browser at: `http://localhost:9000`

```python
import sys
sys.path.append("/path/to/folder/with/visnet") # introduce visnet package

import networkx as nx
import visnet as vn

net = ex.network_states()
net.config['mouse_click_on_bg_action_name'] = 'resetAll'
net.config['mouse_click_on_node_action_name'] = 'neighbourhoodHighlight'
net.config['mouse_click_on_node_action_args'] = 1

heat = ex.heatmap_states()
heat.config['mouse_click_on_bg_action_name'] = 'resetAll'
heat.config['mouse_click_on_cell_action_name'] = 'cellRowHighlight'

com = vn.Compound(1,2)
com.add_visualization(net, 0, 0)
com.add_visualization(heat, 0, 1)

com.render_notebook()
```

Produces:
![Simple Compound](https://imgur.com/IzPzwi5.png "Simple Compound")



### Customization and other features
The available visualizations are highly customizable. Allowing a user to:

- Set properties of nodes, links of `Network` programmatically and statically.
- Set actions on the visualizations.
- Change the navigation mode.
- Make action bindings across different atomic visualizations in a `Compound` visualization.

For demonstrations of these features please refer to the notebooks folder.

**See Also:** [Network Configuration Settings](docs/config_network.md)


## 4. Dependencies

Visnet makes extensive use of:

- [NetworkX](https://networkx.github.io/)
- [Numpy](http://www.numpy.org/)
- [SciPy](https://www.scipy.org/)
- [Numeric Javascript](http://www.numericjs.com/)


## 5. Questions?
This project is still at an early stage of development. A documentation is not available at the moment. For questions and troubleshooting feel free to contact [me](mailto:m.onureken@gmail.com).
