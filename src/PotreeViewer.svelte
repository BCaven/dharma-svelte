<!-- This is the example Potree provides that has a map included-->
<!-- the main thing we need to do to make this work is to change the file paths-->
<script>
	let urlHead = "Potree_1.8";
</script>
<head>
	
    <!-- all of these reference paths need to be changed-->
	<link rel="stylesheet" type="text/css" href="{urlHead}/build/potree/potree.css">
	<link rel="stylesheet" type="text/css" href="{urlHead}/libs/jquery-ui/jquery-ui.min.css">
	<link rel="stylesheet" type="text/css" href="{urlHead}/libs/openlayers3/ol.css">
	<link rel="stylesheet" type="text/css" href="{urlHead}/libs/spectrum/spectrum.css">
	<link rel="stylesheet" type="text/css" href="{urlHead}/libs/jstree/themes/mixed/style.css">
	<link rel="stylesheet" type="text/css" href="{urlHead}/libs/Cesium/Widgets/CesiumWidget/CesiumWidget.css">
</head>

<body>
	<!--<script src="{urlHead}/libs/jquery/jquery-3.1.1.min.js"></script>-->
	<script src="{urlHead}/libs/spectrum/spectrum.js"></script>
	<!--<script src="{urlHead}/libs/jquery-ui/jquery-ui.min.js"></script>-->
	<script src="{urlHead}/libs/other/BinaryHeap.js"></script>
	<script src="{urlHead}/libs/tween/tween.min.js"></script>
	<script src="{urlHead}/libs/d3/d3.js"></script>
	<script src="{urlHead}/libs/proj4/proj4.js"></script>
	<script src="{urlHead}/libs/openlayers3/ol.js"></script>
	<script src="{urlHead}/libs/i18next/i18next.js"></script>
	<script src="{urlHead}/libs/jstree/jstree.js"></script>
	<script src="{urlHead}/build/potree/potree.js"></script>
	<script src="{urlHead}/libs/plasio/js/laslaz.js"></script>
	<script src="{urlHead}/libs/Cesium/Cesium.js"></script>
	
	<!-- INCLUDE ADDITIONAL DEPENDENCIES HERE -->
	<!-- INCLUDE SETTINGS HERE -->
	
	<div class="potree_container" style="position: absolute; width: 100%; height: 100%; left: 0px; top: 0px; ">
		
		<div id="potree_render_area" style="background-image: url('{urlHead}/build/potree/resources/images/background.jpg');">
			<div id="cesiumContainer" style="position: absolute; width: 100%; height: 100%; background-color:green"></div>
		</div>
		
	</div>


<script>

	import * as THREE from urlHead + "/libs/three.js/build/three.module.js"; // probably do not need to import all of this
	
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

	let cp = new Cesium.Cartesian3(4303414.154026048, 552161.235598733, 4660771.704035539); // once again, need to find the correct coordinates for the roman forum (should probably just have it be a variable containing the points so we can edit it later)
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
		Roman forum`);

	potreeViewer.loadGUI(() => {
		potreeViewer.setLanguage('en');
		$("#menu_appearance").next().show();
		$("#menu_tools").next().show();
		$("#menu_scene").next().show();
		potreeViewer.toggleSidebar();
	});
	
	Potree.loadPointCloud("./Potree_1.8/pointclouds/lion_takanawa/cloud.js", "lion", function(e){ // change this url
		let scene = potreeViewer.scene;
		
		scene.addPointCloud(e.pointcloud);
		
		e.pointcloud.position.set(569277.402752, 5400050.599046, 0); // probably will need to change the point cloud starting position
		e.pointcloud.rotation.set(0, 0, -0.035); // also probably needs to be changed

		let material = e.pointcloud.material;
		material.pointSizeType = Potree.PointSizeType.ADAPTIVE;
		material.size = 0.7;
		material.elevationRange = [0, 70];
		material.weightRGB = 1.0;
		material.weightElevation = 1.0;
		
		scene.view.position.set(570975.577, 5398630.521, 1659.311); // these two will need to change
		scene.view.lookAt(570115.285, 5400866.092, 30.009); // this

		{
			let aTownHall = new Potree.Annotation({ // make these the correct annotations (maybe import the old ones?)
				position: [569879.768, 5400886.182, 80.691],
				title: "Town Hall",
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

			{ // Attribute Selector Annotation

				// Create title element with jquery
				let elTitle = $(`
					<span>
						Attribute:
						<img title="Elevation" name="action_elevation" src="${Potree.resourcePath}/icons/profile.svg" class="annotation-action-icon"/>
						<img title="RGB and Elevation" name="action_both" src="${Potree.resourcePath}/icons/rgb_elevation.png" class="annotation-action-icon"/>
						<img title="RGB" name="action_rgb" src="${Potree.resourcePath}/icons/rgb.svg" class="annotation-action-icon"/>
					</span>`);
				elTitle.find("img[name=action_elevation]").click( () => {
					scene.pointclouds.forEach( pc => pc.material.activeAttributeName = "elevation" );
				});
				elTitle.find("img[name=action_rgb]").click( () => {
					scene.pointclouds.forEach( pc => pc.material.activeAttributeName = "rgba" );
				});
				elTitle.find("img[name=action_both]").click( () => {
					scene.pointclouds.forEach( pc => pc.material.activeAttributeName = "composite" );
				});

				// Give the annotation a meaningful string representation for the sidebar
				elTitle.toString = () => "Color Setting";

				// Same as with other annotations, except title is a jquery object this time.
				let aActions = new Potree.Annotation({
					position: [569222.340, 5401213.625, 227],
					title: elTitle
				});
				scene.annotations.add(aActions);
			}

			{ // Attribute Selector Annotation

				let elTitle = $(`
					<span>
						Quality:
						<span name="low"  style="font-family: monospace; margin-left: 4px">low</span>
						<span name="med"  style="font-family: monospace; margin-left: 4px">med</span>
						<span name="high" style="font-family: monospace; margin-left: 4px">high</span>
					</span>`);
				
				elTitle.find("span").mouseover( (e) => {
					$(e.target).css("filter", "drop-shadow(0px 0px 1px white)");
				}).mouseout( (e) => {
					$(e.target).css("filter", "");
				});

				elTitle.find("span[name=low]").click( () => {
					potreeViewer.setPointBudget(1_000_000);
					potreeViewer.useHQ = false;
				});

				elTitle.find("span[name=med]").click( () => {
					potreeViewer.setPointBudget(3_000_000);
					potreeViewer.useHQ = false;
				});

				elTitle.find("span[name=high]").click( () => {
					potreeViewer.setPointBudget(4_000_000);
					potreeViewer.useHQ = true;
				});

				// Give the annotation a meaningful string representation for the sidebar
				elTitle.toString = () => "Quality Setting";

				// Same as with other annotations, except title is a jquery object this time.
				let aActions = new Potree.Annotation({
					position: [570274.902, 5401873.626, 227],
					title: elTitle
				});
				scene.annotations.add(aActions);
			}
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

				let pPos		= new THREE.Vector3(0, 0, 0).applyMatrix4(camera.matrixWorld); // might need to adjust these
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
</body>