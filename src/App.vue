<script>
export default {
    data() {
        return {
            // Граф как список смежности
            graph: {
                u: ["v", "x"],
                v: ["y"],
                x: ["v"],
                y: ["x"],
                w: ["y", "z"],
                z: ["z"],
            },
            // Вершины с координатами для отображения
            nodes: [
                { id: "u", x: 100, y: 50, state: "unvisited" },
                { id: "v", x: 200, y: 150, state: "unvisited" },
                { id: "x", x: 300, y: 50, state: "unvisited" },
                { id: "y", x: 400, y: 150, state: "unvisited" },
                { id: "w", x: 500, y: 50, state: "unvisited" },
                { id: "z", x: 600, y: 150, state: "unvisited" },
            ],
            // Рёбра графа для визуализации
            edges: [],
            visited: {}, // Состояние посещения вершин
            delay: 1000, // Задержка в миллисекундах
        };
    },
    methods: {
        // Инициализация рёбер
        initializeEdges() {
            this.edges = [];
            for (const [from, neighbors] of Object.entries(this.graph)) {
                const fromNode = this.nodes.find((n) => n.id === from);
                neighbors.forEach((to) => {
                    const toNode = this.nodes.find((n) => n.id === to);
                    if (fromNode && toNode) {
                        this.edges.push({
                            id: `${from}->${to}`,
                            x1: fromNode.x + 25,
                            y1: fromNode.y + 25,
                            x2: toNode.x + 25,
                            y2: toNode.y + 25,
                        });
                    }
                });
            }
        },
        // Асинхронная функция DFS
        async dfs(nodeId) {
            // Помечаем текущую вершину как посещённую
            this.visited[nodeId] = true;
            this.updateNodeState(nodeId, "visiting");
            await this.sleep(this.delay);

            // Обходим соседей
            for (const neighbor of this.graph[nodeId]) {
                if (!this.visited[neighbor]) {
                    await this.dfs(neighbor);
                }
            }

            // Завершаем обработку вершины
            this.updateNodeState(nodeId, "finished");
            await this.sleep(this.delay);
        },
        // Запуск обхода DFS
        async startDFS() {
            // Очистка состояния посещения вершин
            this.visited = {};
            // Помечаем все вершины как не посещенные
            this.nodes.forEach((node) => this.updateNodeState(node.id, "unvisited"));
            // Запускаем DFS для всех вершин графа
            for (const node of this.nodes) {
                if (!this.visited[node.id]) {
                    await this.dfs(node.id);
                }
            }
        },
        // Обновление состояния вершины
        updateNodeState(nodeId, state) {
            const node = this.nodes.find((n) => n.id === nodeId);
            if (node) {
                node.state = state;
            }
        },
        // Вспомогательная функция для задержки
        sleep(ms) {
            return new Promise((resolve) => setTimeout(resolve, ms));
        },
    },
    mounted() {
        this.initializeEdges();
    },
};
</script>

<template>
    <div class="container">
        <h1>Graph - Depth-First Search Visualization</h1>
        <div class="graph-container">
            <!-- Вершины графа -->
            <div
                v-for="node in nodes"
                :key="node.id"
                :class="['node', node.state]"
                :style="{ top: node.y + 'px', left: node.x + 'px' }"
            >
                <span class="node__name">{{ node.id }}</span>
            </div>

            <!-- Рёбра графа -->
            <svg class="edges">
                <!-- Определяем стрелки -->
                <defs>
                    <marker id="arrowhead" markerWidth="10" markerHeight="7" refX="10" refY="3.5" orient="auto">
                        <polygon points="0 0, 10 3.5, 0 7" fill="black" />
                    </marker>
                </defs>
                <!-- Отображение рёбер -->
                <line
                    v-for="edge in edges"
                    :key="edge.id"
                    :x1="edge.x1"
                    :y1="edge.y1"
                    :x2="edge.x2"
                    :y2="edge.y2"
                    stroke="black"
                    stroke-width="2"
                    marker-end="url(#arrowhead)"
                />
            </svg>
        </div>
        <div class="btns-wrapper">
            <button class="btn" @click="startDFS">Start DFS</button>
        </div>
    </div>
</template>

<style scoped>
.container {
    text-align: center;
    font-family: Arial, sans-serif;
}

.graph-container {
    position: relative;
    width: 700px;
    height: 300px;
    margin: 0 auto;
    border: 1px solid #ddd;
    background-color: #f9f9f9;
    border-radius: 0.5em;
}

.node {
    position: absolute;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: start;
    font-weight: bold;
    border: 2px solid black;
}

.node.unvisited {
    background-color: white;
}

.node.visiting {
    background-color: lightblue;
}

.node.finished {
    background-color: lightgreen;
}

.edges {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    pointer-events: none;
}

.btns-wrapper {
    margin-top: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
}

.btn {
    padding: 0.75em 1.25em;
    background-color: rgba(69, 188, 109, 1);
    color: #161616;
    border: none;
    border-radius: 0.5em;
    cursor: pointer;
    font-weight: 700;
    font-size: 1em;
    transition: background-color 0.2s ease;
}
.btn:hover {
    background-color: rgba(69, 188, 109, 0.8);
}
</style>
