<script lang="ts">
  import { get, writable } from 'svelte/store';
  import {
    SvelteFlow,
    Controls,
    Background,
    BackgroundVariant,
    MiniMap,
    useSvelteFlow,
    type Node,
    type Edge
  } from '@xyflow/svelte';
  import Sidebar from './Sidebar.svelte';

  import '@xyflow/svelte/dist/style.css';

  const nodes = writable<Node[]>([]);
  const edges = writable<Edge[]>([]);

  let selectedNodeId = writable<string | null>(null);

  function updateNodeLabel(newLabel: string) {
    nodes.update(existingNodes => {
      return existingNodes.map(node => {
        if (node.id === get(selectedNodeId)) {
          return {
            ...node,
            data: {
              ...node.data,
              label: newLabel,
            }
          };
        }
        return node;
      });
    });
  }

  const { screenToFlowPosition } = useSvelteFlow();
  const onDragOver = (event: DragEvent) => {
    event.preventDefault();

    if (event.dataTransfer) {
      event.dataTransfer.dropEffect = 'move';
    }
  };

  const onDrop = (event: DragEvent) => {
    event.preventDefault();

    if (!event.dataTransfer) {
      return null;
    }

    const type = event.dataTransfer.getData('application/svelteflow');

    const position = screenToFlowPosition({
      x: event.clientX,
      y: event.clientY
    });

    const newNode = {
      id: `${Math.random()}`,
      type,
      position,
      data: { label: `${type} node` },
      origin: [0.5, 0.0]
    } satisfies Node;

    nodes.update(existingNodes => [...existingNodes, newNode]);
    selectedNodeId.set(newNode.id);
  };

  const exportToJson = () => {
    const currentNodes = get(nodes);
    const currentEdges = get(edges);

    const graph = {
      nodes: currentNodes,
      edges: currentEdges
    };

    const json = JSON.stringify(graph, null, 2);
    const blob = new Blob([json], { type: 'application/json' });
    const url = URL.createObjectURL(blob);

    const a = document.createElement('a');
    a.href = url;
    a.download = 'graph.json';
    a.click();
    URL.revokeObjectURL(url);
  };

  const importFromJson = async (event: Event) => {
    const input = event.target as HTMLInputElement;
    if (!input.files || input.files.length === 0) {
      return;
    }

    const file = input.files[0];
    const text = await file.text();
    const graph = JSON.parse(text);

    nodes.set(graph.nodes);
    edges.set(graph.edges);
  };

  $: selectedNode = get(nodes).find(node => node.id === get(selectedNodeId));
</script>

<main>
  <SvelteFlow {nodes} {edges} fitView on:dragover={onDragOver} on:drop={onDrop} on:nodeClick={(event) => selectedNodeId.set(event.node.id)}>
    <div class="updatenode__controls">
      <label>label:</label>
      <input value={selectedNode ? selectedNode.data.label : ''} on:input={(evt) => updateNodeLabel(evt.target?.value)} />
    </div>
    <Controls />
    <Background variant={BackgroundVariant.Dots} />
    <MiniMap />
  </SvelteFlow>
  <div>
    <button on:click={exportToJson}>Export to JSON</button>
    <input type="file" accept="application/json" on:change={importFromJson} />
  </div>
  <Sidebar />
</main>

<style>
  main {
    height: 100vh;
    display: flex;
    flex-direction: column-reverse;
  }
  button {
    margin: 1em;
    width: 100px;
  }
  :global(.updatenode__controls) {
    position: absolute;
    right: 10px;
    top: 10px;
    z-index: 4;
    font-size: 12px;
  }

  :global(.updatenode__controls label) {
    display: block;
  }
</style>
