<template>
    <div v-if="isLoading" class="loading">
        <div class="loading-body">
            <img src="/logo.svg" />
            <p>Loading...</p>
        </div>
    </div>

    <div v-if="isAbout" class="about">
        <div class="about-body">
            <h1>О РЕБАР</h1>
            <p>Я сделал этот конфигуратор в качестве примера для демонстрации своих навыков. При разработке сайта использовались Vite, Vue и Three.js, 3D модели сделаны в Blender.</p>
            <!--p>Подробнее об этом проекте читайте по ссылке <a href="https://namazov.online">namazov.online/projects/rebar</a>. Другие работы смотрите на сайте <a href="https://namazov.online">namazov.online</a>.</p-->
            <p>Хотите создать что-то классное? Свяжитесь со мной в телеграме <a href="/">@stanislav_n</a> или пишите на почту <a href="mailto:stanislav.namazov@gmail.com">stanislav.namazov@gmail.com</a></p>
        </div>
    </div>
    
    <div class="container">
        <div class="header">
            <div class="header-logo">
                <img src="/logo.svg" />
                <p>Мебель из арматуры</p>
            </div>
            <button @click="isAbout=!isAbout" :class="{active: isAbout}" class="about-toggle"></button>
        </div>
        <div class="controls" v-if="!isAbout && isOptions">
            
            <div class="control-wrapper control-frame">
                <p>Каркас</p>
                <div v-for="(color, ral, index) in frameColorsPx" v-bind:key="index">
                    <input type="radio" :id="ral" name="frameColor" :value="ral" v-model="currentFrameColor">
                    <label :for="ral" :style="'background-color: #' + color.value.substring(2) + ';'">
                        {{color.title}}
                    </label>
                </div>
            </div>
            
            <div class="control-wrapper control-base">
                <p>Отделка</p>
                <div v-for="(value, key, index) in basesPx" v-bind:key="index">
                    <input type="radio" :id="'bs'+key" name="bases" :value="key" v-model="currentBase">
                    <label :for="'bs'+key">
                        {{value.title}}
                    </label>
                </div>
            </div>
            
            <div class="control-wrapper control-pillow" >
                <p>Подушка</p>
                <div v-for="(value, key, index) in pillowsPx" v-bind:key="index">
                    <input type="radio" :id="'pw'+key" name="pillows" :value="key" v-model="currentPillow">
                    <label :for="'pw'+key">
                        {{value.title}}
                    </label>
                </div>
            </div>

        </div>
        <div ref="canvas"></div>
    </div>
</template>

<script setup>
    import { ref, onMounted} from 'vue'
    import * as THREE from 'three'
    import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
    import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'
    import { RGBELoader } from 'three/addons/loaders/RGBELoader.js'

    const canvas = ref(null)

    let isLoading = ref(true)
    let isAbout = ref(false)
    let isOptions = ref(true)

    const frameColors = {
        ral9005: {
            title: 'RAL 9005',
            value: '0x0e0e10'
        },
        ral7016: {
            title: 'RAL 7016',
            value: '0x383e42'
        },
        ral4010: {
            title: 'RAL 4010',
            value: '0xbc4077'
        },
        ral5023: {
            title: 'RAL 5023',
            value: '0x42698c'
        }
    }
    const frameColorsPx = ref(frameColors)
    let currentFrameColor = ref('ral9005')

    const bases = {
        none: {
            title: 'Нет',
            value: null,
            texture: null,
            pillow_pos: [-0.02,0.68,-0.14]
        },
        rebar: {
            title: 'Арматура',
            value: 'BaseRebar',
            texture: null,
            pillow_pos: [-0.02,0.68,-0.14]
        },
        concrete: {
            title: 'Бетон',
            value: 'BaseConcrete',
            texture: 'concrete',
            metalness: 0,
            roughness: 0.4,
            pillow_pos: [-0.02,0.74,-0.1]
        },
        wood: {
            title: 'Дерево',
            value: 'BaseWood',
            texture: 'wood',
            metalness: 0,
            roughness: 0.4,
            pillow_pos: [-0.02,0.71,-0.12]
        }
    }
    const basesPx = ref(bases)
    let currentBase = ref('wood')

    const pillows = {
        none: {
            title: 'Нет',
            value: null,
        },
        oxford: {
            title: 'Оксфорд',
            value: 'oxford',
            metalness: 0,
            roughness: 0.4
        },
        lather: {
            title: 'Экокожа',
            value: 'lether',
            metalness: 0,
            roughness: 0.3
        },
        textile: {
            title: 'Текстиль',
            value: 'textile',
            metalness: 0,
            roughness: 1
        }
    }
    const pillowsPx = ref(pillows)
    let currentPillow = ref('none')

    onMounted(() => {
        init()
    });

    function init(){
        // create system helpers
        const clock = new THREE.Clock()
        const manager = new THREE.LoadingManager()
        const modelLoader = new GLTFLoader( manager )
        const textureLoader = new THREE.TextureLoader( manager )
        const rgbeLoader = new RGBELoader( manager )
        
        // create renderer
        const renderer = new THREE.WebGLRenderer({ antialias: true })
        renderer.setPixelRatio( window.devicePixelRatio )
        renderer.setSize( window.innerWidth, window.innerHeight )
        renderer.toneMapping = THREE.ACESFilmicToneMapping
        renderer.toneMappingExposure = 0.1
        //renderer.shadowMap.enabled = true
        
        // create scene
        const scene = new THREE.Scene()
        scene.background = new THREE.Color( 0xE9E6E3 );

        // create env
        const envMap = rgbeLoader.load( './textures/royal_esplanade_1k.hdr' );
        envMap.mapping = THREE.EquirectangularReflectionMapping;
        scene.environment = envMap;

        // create camera
        const camera = new THREE.PerspectiveCamera( 90, window.innerWidth / window.innerHeight, 0.1, 1000 )
        camera.rotation.order = 'YXZ'
        camera.position.set(-0.6,1,1.2)

        // create camera controls
        const controls = new OrbitControls( camera, renderer.domElement );
        controls.target.set( 0, 0.5, 0 );
        controls.update();
        controls.enablePan = false;
        controls.enableDamping = true;
        controls.minDistance = 1;
        controls.maxDistance = 5;
        controls.maxPolarAngle = Math.PI / 2;

        // resize renderer when window is resized
        window.onresize = function () {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize( window.innerWidth, window.innerHeight );
        };

        // create lights
        const hemiLight = new THREE.HemisphereLight( 0xddeeff, 0xddeeff);
        hemiLight.position.set( 0, 20, 0 );
        scene.add( hemiLight )

        const light = new THREE.PointLight( 0xffffff, 10, 8 )
        light.position.set( 1, 3, 1 )
        scene.add( light )

        const light2 = new THREE.PointLight( 0xffffff, 10, 8 )
        light2.position.set( -1, 2, 1 )
        scene.add( light2 )

        // create frame textures
        const frameMaterial = {}
        for (const [key, color] of Object.entries(frameColors)) {
            frameMaterial[key] = new THREE.MeshStandardMaterial( { metalness: 0.2, roughness: 0.4, color: parseInt(color.value, 16), name: key } );
        }

        // create pillow textures
        const pillowMaterial = {}
        for (const [key, value] of Object.entries(pillows)) {
            if (key != 'none'){
                pillowMaterial[key] = new THREE.MeshStandardMaterial( { 
                    metalness: value.metalness, 
                    roughness: value.roughness, 
                    name: key, 
                    map: textureLoader.load( './textures/' + value.value + '.jpg' ),
                } );
            }
        }

        // create bases textures
        const baseMaterial = {}
        for (const [key, value] of Object.entries(bases)) {
            if (value.texture != null){
                baseMaterial[key] = new THREE.MeshStandardMaterial( { 
                    metalness: value.metalness, 
                    roughness: value.roughness, 
                    name: key, 
                    map: textureLoader.load( './textures/' + value.texture + '.jpg' ),
                });
            }
        }

        const armchair = {}
        modelLoader.load( './models/armchair.glb', ( file ) => {
            
            
            armchair.frame = file.scene.getObjectByName('Frame')
            
            armchair.base = {}
            for (const [key, value] of Object.entries(bases)) {
                if (value.value != null){
                    armchair.base[key] = file.scene.getObjectByName(value.value)
                    armchair.base[key].material = baseMaterial[value.texture]
                }
            }

            armchair.pillow = file.scene.getObjectByName('Pillow')

            file.scene.visible = true
            file.scene.position.set(0,0.2,0)
            file.scene.rotation.y = -1
            scene.add(file.scene)
        })

        manager.onLoad = async function ( ) {
            isLoading.value = false
            canvas.value.appendChild( renderer.domElement )
            animate()
        }
        
        function animate() {

            for (const [key, value] of Object.entries(armchair.base)) {
                if (armchair.base[currentBase.value] !== undefined){
                    armchair.base[key].visible = false
                    if (currentBase.value != 'none'){
                        armchair.base[currentBase.value].visible = true
                    }
                } else {
                    armchair.base[key].visible = false
                }
                if (currentPillow.value != 'none'){
                    const p = bases[currentBase.value].pillow_pos
                    armchair.pillow.position.set(p[0], p[1], p[2])
                }
            }

            if (armchair.frame.material.name != currentFrameColor.value){
                armchair.frame.material = frameMaterial[currentFrameColor.value]
                armchair.base.rebar.material = frameMaterial[currentFrameColor.value]
            }

            if (currentPillow.value != 'none'){
                armchair.pillow.visible = true
                if (armchair.pillow.material.name != currentPillow.value){
                    armchair.pillow.material = pillowMaterial[currentPillow.value]
                }
            } else {
                armchair.pillow.visible = false
            }

            
            requestAnimationFrame( animate );
            controls.update();
            renderer.render( scene, camera );
        }
    }

</script>

<style>
    @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;900&display=swap');
    *{
        margin: 0;
        padding: 0;
        box-sizing: border-box;
        font-family: 'Montserrat', sans-serif;
        color: black;
    }
    .loading, .about{
        width: 100%;
        height: 100vh;
        display: flex;
        align-items: center;
        justify-content: center;
        position: absolute;
        z-index: 100;
        background: black;
    }
    .loading{
        z-index: 150;
        text-align: center;
        background: white;
    }
    .loading-body .name{
        font-size: 30px;
        font-weight: bold;
    }

    .about-body{
        width: 100%;
        max-width: 700px;
        padding: 30px;
    }
    .about-body *{
        color: white;
    }
    .about-body p{
        margin-top: 30px;
    }

    .container{
        position: relative;
    }

    .header, .controls{
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: space-between;
        position: absolute;
        z-index: 100;
        padding: 30px;
    }

    .header-logo{
        font-size: 12.3px;
    }
    .header-logo img{
        width: 100%;
        max-width: 135px;
    }
    .header-logo .name{
        font-size: 36px;
        font-weight: bold;
        margin-bottom: -7px;
        margin-left: -1px;
    }

    .about-toggle{
        width: 28px;
        height: 28px;
        border: none;
        background: url('./assets/menu.svg') no-repeat center center;
        background-size: cover;
        cursor: pointer;
        transition: all 0.2s;
    }
    .about-toggle.active{
        background: url('./assets/close.svg') no-repeat center center;
        background-size: cover;
    }
    .about-toggle:hover{
        transform: scale(1.1);
    }
    .about-toggle:active{
        transform: scale(0.90);
    }

    .controls{
        align-items: flex-start;
        bottom: 0;
        justify-content: center;
        background: #ffffff70;
        width: auto;
        margin: 20px;
        margin-right: 0;
        font-size: 12px;
    }
    .control-wrapper{
        margin-right: 30px;
    }
    .control-wrapper p{
        margin-bottom: 10px;
    }
    .control-wrapper input{
        display: none;
    }
    .control-wrapper label{
        display: inline-block;
        padding: 5px;
        color: white;
        min-width: 100px;
        margin: 2px;
        background-color: #00000030;
    }
    .control-wrapper input:checked + label{
       border: 2px solid black;
       padding-top: 3px;
       padding-bottom: 3px;
    }

    .control-frame-color{
        display: inline-block;
        width: 20px;
        height: 20px;
    }

</style>