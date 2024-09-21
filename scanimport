import os
import ast
import networkx as nx
import matplotlib.pyplot as plt
from glob import glob
#from networkx.drawing.nx_agraph import write_dot, graphviz_layout # Need to install pygraphviz, and it's badly supported on win .

GLOBPY = "/**/*.py" # Automatically read nested folders.

# Parms.
DIRPROJ = r"C://YOUR/PROJECT/HERE"
FLOUT_PROJ = r"project.dot"
FLOUT_LIB = r"dependencies.dot"
RANK_DIR = "LR" # Graph node general hierarchy direction - up-down, left right etc. ['TB', 'LR', 'BT', 'RL']

# Taken from pyan.
def make_safe_label(label):
    """Avoid name clashes with GraphViz reserved words such as 'graph'."""
    unsafe_words = ("digraph", "graph", "cluster", "subgraph", "node")
    out = label
    for word in unsafe_words:
        out = out.replace(word, "%sX" % word)
    # SBM Added replacement for dot: hyphen -> underline.
    return out.replace(".", "_").replace("*", "").replace("-", "_")

import sys
import io
import logging
class Writer(object):
    def __init__(self, graph, output=None, logger=None, tabstop=4):
        self.graph = graph
        self.output = output
        self.logger = logger or logging.getLogger(__name__)
        self.indent_level = 0
        self.tabstop = tabstop * " "

    def log(self, msg):
        self.logger.info(msg)

    def indent(self, level=1):
        self.indent_level += level

    def dedent(self, level=1):
        self.indent_level -= level

    def write(self, line):
        self.outstream.write(self.tabstop * self.indent_level + line + "\n")

    def run(self):
        self.log("%s running" % type(self))
        try:
            if isinstance(self.output, io.StringIO):  # write to stream
                self.outstream = self.output
            else:
                self.outstream = open(self.output, "w")  # write to file
        except TypeError:
            self.outstream = sys.stdout
        self.start_graph()
        self.write_subgraph(self.graph)
        self.write_edges()
        self.finish_graph()
        if self.output and not isinstance(self.output, io.StringIO):
            self.outstream.close()

    def write_subgraph(self, graph):
        self.start_subgraph(graph)
        for node in graph.nodes:
            self.write_node(node)
        for subgraph in graph.subgraphs:
            self.write_subgraph(subgraph)
        self.finish_subgraph(graph)

    def write_edges(self):
        self.start_edges()
        for edge in self.graph.edges:
            self.write_edge(edge)
        self.finish_edges()

    def start_graph(self):
        pass

    def start_subgraph(self, graph):
        pass

    def write_node(self, node):
        pass

    def start_edges(self):
        pass

    def write_edge(self, edge):
        pass

    def finish_edges(self):
        pass

    def finish_subgraph(self, graph):
        pass

    def finish_graph(self):
        pass

class DotWriter(Writer):
    def __init__(self, graph, options=None, output=None, logger=None, tabstop=4):
        Writer.__init__(self, graph, output=output, logger=logger, tabstop=tabstop)
        options = options or []
        if graph.grouped:
            options += ['clusterrank="local"']
        self.options = ", ".join(options)
        self.grouped = graph.grouped

    def start_graph(self):
        self.write("digraph G {")
        self.write("    graph [" + self.options + "];")
        self.indent()

    def start_subgraph(self, graph):
        self.log("Start subgraph %s" % graph.label)
        # Name must begin with "cluster" to be recognized as a cluster by GraphViz.
        self.write("subgraph cluster_%s {\n" % graph.id)
        self.indent()

        # translucent gray (no hue to avoid visual confusion with any
        # group of colored nodes)
        self.write('graph [style="filled,rounded", fillcolor="#80808018", label="%s"];' % graph.label)

    def finish_subgraph(self, graph):
        self.log("Finish subgraph %s" % graph.label)
        # terminate previous subgraph
        self.dedent()
        self.write("}")

    def write_node(self, node):
        # SBM Simplification. I dunno how the connection works, currently just adds useless nodes.
        # self.write('{} [label="{}", style="filled"];'.format(make_safe_label(node.title()), make_safe_label(node.title())) )
        pass
    
        # self.log("Write node %s" % node.label)
        # self.write(
        #     '%s [label="%s", style="filled", fillcolor="%s",'
        #     ' fontcolor="%s", group="%s"];' % (node.id, node.label, node.fill_color, node.text_color, node.group)
        # )

    def write_edge(self, edge):
        # SBM Simplification.
        source = edge[0]
        target = edge[1]
        self.write('    {} -> {} [style="solid"];'.format(make_safe_label(source), make_safe_label(target)))
        # source = edge.source
        # target = edge.target
        # color = edge.color
        # if edge.flavor == "defines":
        #     self.write('    %s -> %s [style="dashed",  color="%s"];' % (source.id, target.id, color))
        # else:  # edge.flavor == 'uses':
        #     self.write('    %s -> %s [style="solid",  color="%s"];' % (source.id, target.id, color))

    def finish_graph(self):
        self.write("}")  # terminate "digraph G {"

def match_import(lmodules, limports):
    """Match imports to known files list.
    
    Normally, I expect imports to be a partial right match to the files -
    a relative import drops its own nesting.
    I don't know how complex imports would fare.
    Might match ambiguously with multiple files - in which case idc,
     send an edge to all of them. Creep: Should highlight such edges.
    If no files are matched, the likely reason is that this is an external lib.
    """
    lnodes = []
    lextern = []
    for vimport in limports:
        lmatch = [flnm for flnm in lmodules if flnm.endswith(vimport)]
        if len(lmatch) == 0: # External.
            lextern.append(vimport)
        else:
            lnodes.extend(lmatch)
        
    return (lnodes, lextern)

def parse_imports_from_file(file_path):
    """Parse the imports from a single Python file."""
    # if "class_accelerate_launch" in file_path: # TEST
    #     a=1
    with open(file_path, "r", encoding="utf-8") as file:
        tree = ast.parse(file.read(), filename=file_path)

    imports = []
    for node in ast.walk(tree):
        if isinstance(node, ast.Import):
            for alias in node.names:
                imports.append(alias.name)
        elif isinstance(node, ast.ImportFrom):
            if node.module:
                imports.append(node.module)
    
    return imports

def scan_project_files(project_dir):
    """Scan all Python files in a project and map their imports."""
    if os.path.isdir(project_dir):
        scandir = project_dir + GLOBPY
    else:
        scandir =  project_dir
    lfiles = list(glob(scandir, recursive = True))
    # Why the fuck does gepetto keep using this crap.
    # for root, _, files in os.walk(project_dir):
    #     for file in files:
    #         if file.endswith(".py"):
    #            file_path = os.path.join(root, file)
    dmodules = dict()
    for file_path in lfiles:
        # Find what the file should be called in import syntax.
        relative_path = os.path.relpath(file_path, project_dir)
        modname = os.path.splitext(relative_path)[0] # Remove extension.
        modname = os.path.normpath(modname).split(os.sep) # Split all types of path delims.
        modname = ".".join(modname) # Reconstruct with periods.
        dmodules[file_path] = modname
    lmodules = list(dmodules.values())
        
    dproject = dict()
    ddepend = dict()
    for file_path in lfiles:
        limports = parse_imports_from_file(file_path)
        (lnodes, lextern) = match_import(lmodules, limports)
        dproject[dmodules[file_path]] = lnodes
        ddepend[dmodules[file_path]] = lextern
    return (dproject, ddepend)

# Jiminy
def create_dependency_graph(project_files):
    """Create a dependency graph using networkx."""
    G = nx.DiGraph()

    # SBM Cull keys which lead nowhere, likely external modules.
    # project_only = {k:[imp for imp in v if imp in project_files.keys()]
    #                 for k,v in project_files.items()}
    for file, imports in project_files.items():
        G.add_node(file)
        for imp in imports:
            G.add_edge(file, imp)

    # SBM Adding props per pyan's reqs.
    G.grouped = False
    G.label = "Whatever"
    G.id = 1234
    G.subgraphs = []

    return G
    
# Jiminy.
def visualize_graph(graph):
    """Draws the given dependency graph.

    Args:
        G: A NetworkX DiGraph object representing the dependency graph.
    """

    pos = nx.spring_layout(graph) # Circle bullship.
    # pos = nx.multipartite_layout(graph) # Valuerror: all nodes must have subset_key as data.
    # pos = graphviz_layout(graph, prog = 'dot')
    nx.draw(graph, pos, with_labels=True, node_size=500, font_size=10)
    plt.show()

def main():
    """Main."""
    project_dir = DIRPROJ # Project root.
    (dproject,ddepend) = scan_project_files(project_dir)
    # Create dot graph for project.
    dep_graph = create_dependency_graph(dproject)
    # visualize_graph(dep_graph)
    writer = DotWriter(dep_graph, output = FLOUT_PROJ, options=["rankdir=" + RANK_DIR])
    #, options=["rankdir=" + known_args.rankdir], output=known_args.filename, logger=logger)
    writer.run()
    # Create dot graph for dependencies.
    dep_graph = create_dependency_graph(ddepend)
    # visualize_graph(dep_graph)
    writer = DotWriter(dep_graph, output = FLOUT_LIB, options=["rankdir=" + RANK_DIR])
    writer.run()

# Main
if __name__ == "__main__":
    main()
    print("FIN")
    
