<script>
	let urlHead = "./Potree_1.8";
	import "./Potree_1.8/libs/jquery/jquery-3.1.1.min.js";
	import "./Potree_1.8/libs/spectrum/spectrum.js";
	import "./Potree_1.8/libs/jquery-ui/jquery-ui.min.js";
	import "./Potree_1.8/libs/other/BinaryHeap.js";
	import "./Potree_1.8/libs/tween/tween.min.js";
	import "./Potree_1.8/libs/d3/d3.js";
	import "./Potree_1.8/libs/proj4/proj4.js";
	import "./Potree_1.8/libs/openlayers3/ol.js";
	import "./Potree_1.8/libs/i18next/i18next.js";
	import "./Potree_1.8/libs/jstree/jstree.js";
	import "./Potree_1.8/build/potree/potree.js";
	import "./Potree_1.8/libs/plasio/js/laslaz.js";
	import "./Potree_1.8/libs/Cesium/Cesium.js";
	import * as THREE from "./Potree_1.8/libs/three.js/build/three.module.js";
	
	// make struct (or whatever the js equivalent is) to create and store annotations
	function Annotation(position, name, camera_loc, camera_tar) {
		self.pos = position; // coordinates are arrays of three floats
		self.name = name;
		self.cam_loc = camera_loc;
		self.cam_tar = camera_tar;
	}

	let annotations = [
		new Annotation([100, 100, 100], "test", [120, 120, 120], [100, 100, 100])
	]
	window.cesiumViewer = new Cesium.Viewer('cesiumContainer', {
		useDefaultRenderLoop: false,
		animation: false,
		baseLayerPicker : false,
		fullscreenButton: false, 
		geocoder: false,
		homeButton: false,
		infoBox: false,
		sceneModePicker: false,
		selectionIndicator: false,
		timeline: false,
		navigationHelpButton: false,
		imageryProvider : Cesium.createOpenStreetMapImageryProvider({url : 'https://a.tile.openstreetmap.org/'}),
		terrainShadows: Cesium.ShadowMode.DISABLED,
	});

	let cp = new Cesium.Cartesian3(4303414.154026048, 552161.235598733, 4660771.704035539);
	cesiumViewer.camera.setView({
		destination : cp,
		orientation: {
			heading : 10, 
			pitch : -Cesium.Math.PI_OVER_TWO * 0.5, 
			roll : 0.0 
		}
	});

	window.potreeViewer = new Potree.Viewer(document.getElementById("potree_render_area"), {
		useDefaultRenderLoop: false
	});
	potreeViewer.setEDLEnabled(true);
	potreeViewer.setFOV(60);
	potreeViewer.setPointBudget(3_000_000);
	potreeViewer.setMinNodeSize(50);
	potreeViewer.loadSettingsFromURL();
	potreeViewer.setBackground(null);
	potreeViewer.useHQ = true;

	potreeViewer.setDescription(`
		Potree using <a href="https://cesiumjs.org/" target="_blank">Cesium</a> to display an 
		<a href="https://www.openstreetmap.org" target="_blank">OpenStreetMap</a> map below.<br>
		Point cloud courtesy of <a href="http://riegl.com/" target="_blank">Riegl</a><br>`);

	potreeViewer.loadGUI(() => {
		potreeViewer.setLanguage('en');
		//$("#menu_appearance").next().show();
		//$("#menu_tools").next().show();
		//$("#menu_scene").next().show();
		//potreeViewer.toggleSidebar();
	});
	
	Potree.loadPointCloud("http://5.9.65.151/mschuetz/potree/resources/pointclouds/riegl/retz/cloud.js", "Retz", function(e){
		let scene = potreeViewer.scene;
		
		scene.addPointCloud(e.pointcloud);
		
		e.pointcloud.position.set(569277.402752, 5400050.599046, 0);
		e.pointcloud.rotation.set(0, 0, -0.035);

		let material = e.pointcloud.material;
		material.pointSizeType = Potree.PointSizeType.ADAPTIVE;
		material.size = 0.7;
		material.elevationRange = [0, 70];
		material.weightRGB = 1.0;
		material.weightElevation = 1.0;
		
		scene.view.position.set(570975.577, 5398630.521, 1659.311);
		scene.view.lookAt(570115.285, 5400866.092, 30.009);
		
		for (var anno in annotations) {
			let tempAnno = new Potree.Annotation({
				position: anno.pos,
				title: anno.name,
				cameraPosition: anno.cam_loc,
				cameraTarget: anno.cam_tar
			});
			scene.annotations.add(tempAnno);
			/*
			let aTownHall = new Potree.Annotation({
				position: [569879.768, 5400886.182, 80.691],
				title: "Temple of Vespain and ",
				cameraPosition: [569955.329, 5400822.949, 98.807],
				cameraTarget: [569879.768, 5400886.182, 46.691]
			});
			scene.annotations.add(aTownHall);

			let aTrainStation = new Potree.Annotation({
				position: [570337.407, 5400522.730, 30],
				title: "Train Station",
				cameraPosition: [570377.074, 5400427.884, 100.576],
				cameraTarget: [570337.407, 5400522.730, 18.595]
			});
			scene.annotations.add(aTrainStation);
			*/

		}
		

		//let pointcloudProjection = e.pointcloud.projection;
		let pointcloudProjection = "+proj=utm +zone=33 +ellps=WGS84 +datum=WGS84 +units=m +no_defs";
		let mapProjection = proj4.defs("WGS84");

		window.toMap = proj4(pointcloudProjection, mapProjection);
		window.toScene = proj4(mapProjection, pointcloudProjection);
		
		{
			let bb = potreeViewer.getBoundingBox();

			let minWGS84 = proj4(pointcloudProjection, mapProjection, bb.min.toArray());
			let maxWGS84 = proj4(pointcloudProjection, mapProjection, bb.max.toArray());
		}
	});

	function loop(timestamp){
		requestAnimationFrame(loop);

		potreeViewer.update(potreeViewer.clock.getDelta(), timestamp);

		potreeViewer.render();

		if(window.toMap !== undefined){

			{
				let camera = potreeViewer.scene.getActiveCamera();

				let pPos		= new THREE.Vector3(0, 0, 0).applyMatrix4(camera.matrixWorld);
				let pRight  = new THREE.Vector3(600, 0, 0).applyMatrix4(camera.matrixWorld);
				let pUp		 = new THREE.Vector3(0, 600, 0).applyMatrix4(camera.matrixWorld);
				let pTarget = potreeViewer.scene.view.getPivot();

				let toCes = (pos) => {
					let xy = [pos.x, pos.y];
					let height = pos.z;
					let deg = toMap.forward(xy);
					let cPos = Cesium.Cartesian3.fromDegrees(...deg, height);

					return cPos;
				};

				let cPos = toCes(pPos);
				let cUpTarget = toCes(pUp);
				let cTarget = toCes(pTarget);

				let cDir = Cesium.Cartesian3.subtract(cTarget, cPos, new Cesium.Cartesian3());
				let cUp = Cesium.Cartesian3.subtract(cUpTarget, cPos, new Cesium.Cartesian3());

				cDir = Cesium.Cartesian3.normalize(cDir, new Cesium.Cartesian3());
				cUp = Cesium.Cartesian3.normalize(cUp, new Cesium.Cartesian3());

				cesiumViewer.camera.setView({
					destination : cPos,
					orientation : {
						direction : cDir,
						up : cUp
					}
				});
				
			}

			let aspect = potreeViewer.scene.getActiveCamera().aspect;
			if(aspect < 1){
				let fovy = Math.PI * (potreeViewer.scene.getActiveCamera().fov / 180);
				cesiumViewer.camera.frustum.fov = fovy;
			}else{
				let fovy = Math.PI * (potreeViewer.scene.getActiveCamera().fov / 180);
				let fovx = Math.atan(Math.tan(0.5 * fovy) * aspect) * 2
				cesiumViewer.camera.frustum.fov = fovx;
			}
			
		}

		cesiumViewer.render();
	}

	requestAnimationFrame(loop);



</script>
<head>
	<meta charset="utf-8">
	<meta name="description" content="">
	<meta name="author" content="">
	<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
	<title>Potree Viewer</title>

	<link rel="stylesheet" type="text/css" href="../build/potree/potree.css">
	<link rel="stylesheet" type="text/css" href="../libs/jquery-ui/jquery-ui.min.css">
	<link rel="stylesheet" type="text/css" href="../libs/openlayers3/ol.css">
	<link rel="stylesheet" type="text/css" href="../libs/spectrum/spectrum.css">
	<link rel="stylesheet" type="text/css" href="../libs/jstree/themes/mixed/style.css">
	<link rel="stylesheet" type="text/css" href="../libs/Cesium/Widgets/CesiumWidget/CesiumWidget.css">
</head>

<body>

	
	
	<div class="potree_container" style="position: absolute; width: 100%; height: 100%; left: 0px; top: 0px; ">
		
		<div id="potree_render_area" style="background-image: url('{urlHead}/build/potree/resources/images/background.jpg');">
			<div id="cesiumContainer" style="position: absolute; width: 100%; height: 100%; background-color:green"></div>
		</div>
		<div id="potree_sidebar_container"> </div>
	</div>



</body>
