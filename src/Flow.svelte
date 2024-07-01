<script lang="ts">
  import { writable } from 'svelte/store';
  import {
    SvelteFlow,
    Controls,
    Background,
    BackgroundVariant,
    MiniMap,
    useSvelteFlow,
    type Node
  } from '@xyflow/svelte';
  import Sidebar from './Sidebar.svelte';

  import '@xyflow/svelte/dist/style.css';

  //const nodes = writable<Node[]>([]);
  
  const nodes = writable<Node[]>([
    {
      id: '1',
      data: { label: 'Input Node' },
      position: { x: 150, y: 5 }
    },

  ]);

  const edges = writable([
    {
      id: '1-2',
      source: '1',
      target: '2'
    }
  ]);

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

    $nodes.push(newNode);
    $nodes = $nodes;
  };
</script>

<main>
  <SvelteFlow {nodes} {edges} fitView on:dragover={onDragOver} on:drop={onDrop}>
    <Controls />
    <Background variant={BackgroundVariant.Dots} />
    <MiniMap />
  </SvelteFlow>
  <Sidebar />
</main>

<style>
  main {
    height: 100vh;
    display: flex;
    flex-direction: column-reverse;
  }
</style>
