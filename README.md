# Project dependency graph
Interactive graph for project and external imports in python.

I have tried pydeps, snakefood, sourcetrail, import-deps and pyan.
Found none to be satisfactory - some for old versions and abandoned, some difficult to configure, some so bloated that without premature filtering it's impossible to understand what's going on in the graph.

What this repo does:
- Scans imports / from only, rather than objects and methods, reducing the number of nodes & edges significantly.
- Glob is used to scan all nested files.
- Matches relative imports to the full name.
- Splits the imports to project graph and external graph - external imports are ones which do not appear to exist in the project directory. This further reduces the clutter, as each graph is structed differently (tree vs bush) and serves a different purpose.
- Creates a simple dot graph (based on [pyan's writer class and module](https://github.com/Technologicat/pyan)).
- An interactive standalone html visualiser is included which reads output graphs and highlights on hover / click nodes and their neighbours - this makes tracing a dependency path much easier without much prior knowledge.

Examples (from [bmaltais' kohya repo](https://github.com/bmaltais/kohya_ss/)):

![Visualise1E](https://github.com/user-attachments/assets/803c40d0-1c17-4061-9200-10160a409b22)

![Visualise2E](https://github.com/user-attachments/assets/437a98eb-b0e6-4c47-8034-7ce747bd5c75)