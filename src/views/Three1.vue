<template>
    <div>
        <div id="container"></div>
        <div class="controls-box">
            <section>
                <button @click="loadJSON">loadJSON</button>
                <button @click="saveToJSON">saveToJSON</button>
            </section>
        </div>
        <input type="file" placeholder="" placeholder-class="input-placeholder" @change="fileChange" />
        <button @click="loadJSON1">loadJSON1

        </button>
    </div>
</template>

<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
export default {
    data() {
        return {
            properties: {
                radius: {
                    name: 'radius',
                    value: 18,
                    min: 0,
                    max: 60,
                    step: 1
                },
                tube: {
                    name: 'tube',
                    value: 2,
                    min: 0,
                    max: 20,
                    step: 0.1
                },
                radialSegments: {
                    name: 'radialSegments',
                    value: 230,
                    min: 0,
                    max: 400,
                    step: 1
                },
                tubularSegments: {
                    name: 'tubularSegments',
                    value: 8,
                    min: 0,
                    max: 50,
                    step: 1
                },
                p: {
                    name: 'p',
                    value: 4,
                    min: 0,
                    max: 20,
                    step: 1
                },
                q: {
                    name: 'q',
                    value: 3,
                    min: 0,
                    max: 20,
                    step: 1
                }
            },
            mesh: null,
            camera: null,
            scene: null,
            renderer: null,
            controls: null
        }
    },
    mounted() {
        this.init()
    },
    methods: {
        formatTooltip(val) {
            return val
        },
        // 初始化
        init() {
            this.createScene() // 创建场景
            this.createMesh() // 创建网格模型
            this.createLight() // 创建光源
            this.createCamera() // 创建相机
            this.createRender() // 创建渲染器
            this.createControls() // 创建控件对象
            this.render() // 渲染
        },
        // 创建场景
        createScene() {
            this.scene = new THREE.Scene()
        },
        // 创建网格模型
        createMesh() {
            // 创建环面扭结
            const geom = new THREE.TorusKnotGeometry(
                this.properties.radius.value,
                this.properties.tube.value,
                this.properties.radialSegments.value,
                this.properties.tubularSegments.value,
                this.properties.p.value,
                this.properties.q.value
            )
            // 创建材质
            const meshMaterial = new THREE.MeshBasicMaterial({
                vertexColors: THREE.VertexColors,
                wireframe: true,
                wireframeLinewidth: 2,
                color: 0xaaaaaa
            })

            // 添加组合材质
            this.mesh = new THREE.Mesh(geom, meshMaterial)

            // 网格对象添加到场景中
            this.scene.add(this.mesh)
        },
        // 网格对象保存为json格式
        saveToJSON() {
            const json = this.mesh.toJSON()
            localStorage.setItem('json', JSON.stringify(json))
        },
        // 导入json格式的网格对象
        loadJSON() {
            this.removeMesh()
            const json = localStorage.getItem('json')

            if (json) {
                console.log('loadJSON', json)
                const loadedGeometry = JSON.parse(json)
                const loader = new THREE.ObjectLoader()
                const loadedMesh = loader.parse(loadedGeometry)
                this.scene.add(loadedMesh)
            }
        },
        loadJSON1() {
            this.removeMesh()
            fetch('http://localhost:5173/models/app.json')
                .then(res => res.json())
                .then(json => {
                    debugger
                    const loader = new THREE.ObjectLoader()
                    const loadedMesh = loader.parse(json.scene)
                    // const loader = new THREE.ObjectLoader()
                    // const loadedMesh = loader.parse(json)
                    this.scene.add(loadedMesh)
                })

        },
        removeMesh() {
            const toRemove = []
            this.scene.traverse(e => {
                if (e instanceof THREE.Mesh) toRemove.push(e)
            })
            toRemove.forEach(e => {
                this.scene.remove(e)
            })
        },
        // 创建光源
        createLight() {
            // 环境光
            const ambientLight = new THREE.AmbientLight(0xffffff, 0.1) // 创建环境光
            this.scene.add(ambientLight) // 将环境光添加到场景

            const spotLight = new THREE.SpotLight(0xffffff) // 创建聚光灯
            spotLight.position.set(-40, 60, -10)
            spotLight.castShadow = true
            this.scene.add(spotLight)
        },
        // 创建相机
        createCamera() {
            const element = document.getElementById('container')
            const width = element.clientWidth // 窗口宽度
            const height = element.clientHeight // 窗口高度
            const k = width / height // 窗口宽高比
            // PerspectiveCamera( fov, aspect, near, far )
            this.camera = new THREE.PerspectiveCamera(35, k, 0.1, 1000)
            this.camera.position.set(-80, 60, 40) // 设置相机位置

            this.camera.lookAt(new THREE.Vector3(10, 0, 0)) // 设置相机方向
            this.scene.add(this.camera)
        },
        // 创建渲染器
        createRender() {
            const element = document.getElementById('container')
            this.renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
            this.renderer.setSize(element.clientWidth, element.clientHeight) // 设置渲染区域尺寸
            this.renderer.shadowMap.enabled = true // 显示阴影
            this.renderer.shadowMap.type = THREE.PCFSoftShadowMap
            this.renderer.setClearColor(0x3f3f3f, 1) // 设置背景颜色
            element.appendChild(this.renderer.domElement)
        },

        // 更新
        redraw() {
            this.removeMesh()
            this.createMesh()
        },
        render() {
            this.renderer.render(this.scene, this.camera)
            requestAnimationFrame(this.render)
        },
        // 创建控件对象
        createControls() {
            this.controls = new OrbitControls(this.camera, this.renderer.domElement)
        },
        fileChange(event) {
            const file = event.target.files[0]
            const reader = new FileReader()
            reader.readAsText(file)
            reader.onload = () => {
                debugger;
                const json = JSON.parse(reader.result)
                const loader = new THREE.ObjectLoader()
                const loadedMesh = loader.parse(json)
                this.scene.add(loadedMesh)
            }

        }
    }
}
</script>
<style>
#container {
    position: absolute;
    min-width: 500px;
    min-height: 500px;
    width: 100%;
    height: 100%;
}

.controls-box {
    position: absolute;
    right: 5px;
    top: 5px;
    width: 300px;
    padding: 10px;
    background-color: #fff;
    border: 1px solid #c3c3c3;
}

.vertice-span {
    line-height: 38px;
    padding: 0 2px 0 10px;
}
</style>