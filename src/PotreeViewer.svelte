<!-- This is the example Potree provides that has a map included-->
<head>
	<meta charset="utf-8">
	<meta name="description" content="">
	<meta name="author" content="">
	<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
	<title>Potree Viewer</title>

    <!-- make sure these file locations are right, right now they are probably not -->
	<link rel="stylesheet" type="text/css" href="../../build/potree/potree.css">
	<link rel="stylesheet" type="text/css" href="../../libs/jquery-ui/jquery-ui.min.css">
	<link rel="stylesheet" type="text/css" href="../../libs/openlayers3/ol.css">
	<link rel="stylesheet" type="text/css" href="../../libs/spectrum/spectrum.css">
	<link rel="stylesheet" type="text/css" href="../../libs/jstree/themes/mixed/style.css">
</head>

<body>
    <!-- We will need to change almost all of these file locations to match... fuck-->
	<script src="../../libs/jquery/jquery-3.1.1.min.js"></script> 
	<script src="../../libs/spectrum/spectrum.js"></script>
	<script src="../../libs/jquery-ui/jquery-ui.min.js"></script>
	
	<script src="../../libs/other/BinaryHeap.js"></script>
	<script src="../../libs/tween/tween.min.js"></script>
	<script src="../../libs/d3/d3.js"></script>
	<script src="../../libs/proj4/proj4.js"></script>
	<script src="../../libs/openlayers3/ol.js"></script>
	<script src="../../libs/i18next/i18next.js"></script>
	<script src="../../libs/jstree/jstree.js"></script>
	<script src="../../build/potree/potree.js"></script>
	<script src="../../libs/plasio/js/laslaz.js"></script>
	
	<!-- INCLUDE ADDITIONAL DEPENDENCIES HERE -->
	<!-- INCLUDE SETTINGS HERE -->
	
	<div class="potree_container" style="position: absolute; width: 100%; height: 100%; left: 0px; top: 0px; ">
		<div id="potree_render_area"></div>
		<div id="potree_sidebar_container"> </div>
	</div>
	
	<script>

	import * as THREE from "../../libs/three.js/build/three.module.js";
	
		window.viewer = new Potree.Viewer(document.getElementById("potree_render_area"));
		
		viewer.setEDLEnabled(true);
		viewer.setFOV(60);
		viewer.setPointBudget(3_000_000);
		viewer.loadSettingsFromURL();
		
		//viewer.setDescription('Point cloud courtesy of <a href="http://riegl.com/" target="_blank">Riegl</a><br>');
		
		viewer.loadGUI(() => {
			viewer.setLanguage('en');
			$("#menu_tools").next().show();
			//viewer.toggleSidebar();
		});
		
		// Load and add point cloud to scene
		Potree.loadPointCloud("http://5.9.65.151/mschuetz/potree/resources/pointclouds/riegl/retz/cloud.js", "Retz", function(e){ // replace this
			let scene = viewer.scene;
			let pointcloud = e.pointcloud;
			
			scene.addAnnotation([31.76, 0.67, 38.50], { // change these
				cameraPosition: [81.91, -89.66, 95.98],
				cameraTarget: [31.76, 0.67, 28.50],
				title: "Town Square"
			});
			
			scene.addAnnotation([451.31, -363.71, 27.40], { // change these
				cameraPosition: [387.13, -382.97, 53.70],
				cameraTarget: [451.31, -363.71, 17.40],
				title: "Train Station"
			});
			
			let material = pointcloud.material;
			material.size = 1;
			material.pointSizeType = Potree.PointSizeType.ADAPTIVE;
			material.shape = Potree.PointShape.SQUARE;
			
			scene.addPointCloud(pointcloud);
			
			e.pointcloud.position.z = 0;
			
            // we will want to keep track of the original THREE vectors that we set the camera to so we can revert to that position with the "reset" button
			scene.view.position.set(519.55, -1062.47, 723.01); // probably change these
			scene.view.lookAt(new THREE.Vector3(164.68, -106.56, 0));
		});
		
	</script>
	
	
  </body>