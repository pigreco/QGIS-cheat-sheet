
# QGIS 3 cheat sheet

Cheat sheet for PyQgis

## Table of Contents

- [UI](#ui)
- [Canvas](#canvas)
- [Decorators](#decorators)
- [Processing algorithms](#processing-algorithms)
- [TOC](#toc)
- [Advanced TOC](#advanced-toc)
- [Layers](#layers)
- [Settings](#settings)
- [ToolBars](#toolbars)
- [Menus](#menus)
- [Common PyQGIS functions](#common-pyqgis-functions)
- [Sources](#sources)


UI
---
__Change Look & Feel__

	app = QApplication.instance()
	qss_file = open(r"style.qss").read()
	app.setStyleSheet(qss_file)

__Change Icon and Title__

	icon = QIcon(r"logo.png")
	iface.mainWindow().setWindowIcon(icon)	
	iface.mainWindow().setWindowTitle("CNE")

Canvas
---
__Access Canvas__

	canvas = iface.mapCanvas()

__Change Canvas color__

	iface.mapCanvas().setCanvasColor(QtCore.Qt.black)		
	iface.mapCanvas().refresh()

&uparrow; [Back to top](#table-of-contents)

Decorators
---
__CopyRight__

	from PyQt5.QtCore import *
	from PyQt5.QtGui import *

	mQFont = "Sans Serif"
	mQFontsize = 9
	mLabelQString = "© QGIS 2019"
	mMarginHorizontal = 0
	mMarginVertical = 0
	mLabelQColor = "#FF0000"

	INCHES_TO_MM = 0.0393700787402
	case = 2

	def AddCopyRight(p,text,xOffset,yOffset):
	    p.translate( xOffset , yOffset  )
	    text.drawContents(p)
	    p.setWorldTransform( p.worldTransform() )

	def _onRenderComplete(p):
	    deviceHeight = p.device().height()
	    deviceWidth  = p.device().width()
	    text = QTextDocument()
	    font = QFont()
	    font.setFamily(mQFont)
	    font.setPointSize(int(mQFontsize))
	    text.setDefaultFont(font)
	    style = "<style type=\"text/css\"> p {color: " + mLabelQColor + "}</style>"
	    text.setHtml( style + "<p>" + mLabelQString + "</p>" )
	    size = text.size()

	    # RenderMillimeters
	    pixelsInchX  = p.device().logicalDpiX()
	    pixelsInchY  = p.device().logicalDpiY()
	    xOffset  = pixelsInchX  * INCHES_TO_MM * int(mMarginHorizontal)
	    yOffset  = pixelsInchY  * INCHES_TO_MM * int(mMarginVertical)

	    if case == 0:
		# Top Left
		AddCopyRight(p, text, xOffset, yOffset)

	    elif case == 1:
		# Bottom Left
		yOffset = deviceHeight - yOffset - size.height()
		AddCopyRight(p, text, xOffset, yOffset)

	    elif case == 2:
		# Top Right
		xOffset  = deviceWidth  - xOffset - size.width()
		AddCopyRight(p, text, xOffset, yOffset)

	    elif case == 3: 
		# Bottom Right
		yOffset  = deviceHeight - yOffset - size.height()
		xOffset  = deviceWidth  - xOffset - size.width()
		AddCopyRight(p, text, xOffset, yOffset)

	    elif case == 4:
		# Top Center
		xOffset = deviceWidth / 2
		AddCopyRight(p, text, xOffset, yOffset)
	    else:
		# Bottom Center
		yOffset = deviceHeight - yOffset - size.height()
		xOffset = deviceWidth / 2
		AddCopyRight(p, text, xOffset, yOffset)


	iface.mapCanvas().renderComplete.connect(_onRenderComplete)
	iface.mapCanvas().refresh()


&uparrow; [Back to top](#table-of-contents)

Processing algorithms 
---

__Get algorithms list__

	for alg in QgsApplication.processingRegistry().algorithms():
		print("{}:{} --> {}".format(alg.provider().name(), alg.name(), alg.displayName()))
	
	# or 
	
	def alglist():
	  s = ''
	  for i in QgsApplication.processingRegistry().algorithms():
	    l = i.displayName().ljust(50, "-")
	    r = i.id()
	    s += '{}--->{}\n'.format(l, r)
	  print(s)

	alglist()

__Get algorithms Help__

Random selection

	import processing
	processing.algorithmHelp("qgis:randomselection")


__How many algorithms are there?__

	len(QgsApplication.processingRegistry().algorithms())

__How many providers are there?__

	len(QgsApplication.processingRegistry().providers())

__How many Expressions are there?__

	len(QgsExpression.Functions()) 
	
&uparrow; [Back to top](#table-of-contents)

TOC
---

__Access checked Layers__

	iface.mapCanvas().layers()

__Obtain Layers name__

	canvas = iface.mapCanvas()
	layers = [canvas.layer(i) for i in range(canvas.layerCount())]
	layers_names = [ layer.name() for layer in layers ]
	print "layers TOC = ", layers_names
	
	or
	
	layers = [layer for layer in QgsProject.instance().mapLayers().values()]


__Add vector layer__

	layer = iface.addVectorLayer("input.shp", "name", "ogr")
	if not layer:
	  print "Layer failed to load!"

__Find layer by name__

	from qgis.core import QgsProject
	layer = QgsProject.instance().mapLayersByName("name")[0]
	print layer.name()

__Set Active layer__

	from qgis.core import QgsProject
	layer = QgsProject.instance().mapLayersByName("name")[0]
	iface.setActiveLayer(layer)

__Remove all layers__

	QgsProject.instance().removeAllMapLayers()

__Remove Contextual menu__

	ltv = iface.layerTreeView()
	ltv.setMenuProvider( None )	

__See the CRS__

	for layer in QgsProject().instance().mapLayers().values():   
	    crs = layer.crs().authid()
	    layer.setName(layer.name() + ' (' + crs + ')')

__Set the CRS__

	for layer in QgsProject().instance().mapLayers().values():
	    layer.setCrs(QgsCoordinateReferenceSystem(4326, QgsCoordinateReferenceSystem.EpsgCrsId))

__Load all layers from GeoPackage__

	fileName = "sample.gpkg"
	layer = QgsVectorLayer(fileName,"test","ogr")
	subLayers =layer.dataProvider().subLayers()

	for subLayer in subLayers:
		name = subLayer.split('!!::!!')[1]
		uri = "%s|layername=%s" % (fileName, name,)
		#Create layer
		sub_vlayer = QgsVectorLayer(uri, name, 'ogr')
		#Add layer to map
		QgsProject.instance().addMapLayer(sub_vlayer)

__Load tile layer (XYZ-Layer)__

	def loadXYZ(url, name):
		rasterLyr = QgsRasterLayer("type=xyz&url=" + url, name, "wms")
		QgsProject.instance().addMapLayer(rasterLyr)

	urlWithParams = 'type=xyz&url=https://a.tile.openstreetmap.org/%7Bz%7D/%7Bx%7D/%7By%7D.png&zmax=19&zmin=0&crs=EPSG3857'
	loadXYZ(urlWithParams, 'OpenStreetMap')

&uparrow; [Back to top](#table-of-contents)


Advanced TOC
---

__Root node__

	root = QgsProject.instance().layerTreeRoot()
	print (root)
	print (root.children())
	
__Access the first child node__

	child0 = root.children()[0]
	print (child0)
	print (type(child0))
	print (isinstance(child0, QgsLayerTreeLayer))
	print (child0.parent())

__Find groups and nodes__

	for child in root.children():
	  if isinstance(child, QgsLayerTreeGroup):
	    print ("- group: " + child.name())
	  elif isinstance(child, QgsLayerTreeLayer):
	    print ("- layer: " + child.name() + "  ID: " + child.layerId())
    
__Find group by name__

	print (root.findGroup("Name"))

__Add layer__

	layer1 = QgsVectorLayer("Point?crs=EPSG:4326", "Layer 1", "memory")
	QgsProject.instance().addMapLayer(layer1, False)
	node_layer1 = root.addLayer(layer1)

__Add Group__

	node_group2 = QgsLayerTreeGroup("Group 2")
	root.addChildNode(node_group2)

__Add Node__
	root.removeChildNode(node_group2)
	root.removeLayer(layer1)

__Move Node__

	cloned_group1 = node_group1.clone()
	root.insertChildNode(0, cloned_group1)
	root.removeChildNode(node_group1)
	
__Rename None__

	node_group1.setName("Group X")
	node_layer2.setName("Layer X")

__Changing visibility__

	print (node_group1.isVisible())
	node_group1.setItemVisibilityChecked(False)
	node_layer2.setItemVisibilityChecked(False)

__Expand Node__

	print (node_group1.isExpanded())
	node_group1.setExpanded(False)

__Hidden Node Trick__

	model = iface.layerTreeView().layerTreeModel()
	ltv = iface.layerTreeView()
	root = QgsProject.instance().layerTreeRoot()

	layer = QgsProject.instance().mapLayersByName(u'Name')[0]
	node=root.findLayer( layer.id())

	index = model.node2index( node )
	ltv.setRowHidden( index.row(), index.parent(), True )
	node.setCustomProperty( 'nodeHidden', 'true')
	ltv.setCurrentIndex(model.node2index(root))  

__Node Signals__

	def onWillAddChildren(node, indexFrom, indexTo):
	  print ("WILL ADD", node, indexFrom, indexTo)

	def onAddedChildren(node, indexFrom, indexTo):
	  print ("ADDED", node, indexFrom, indexTo)

	root.willAddChildren.connect(onWillAddChildren)
	root.addedChildren.connect(onAddedChildren)

__Create new TOC__

	from qgis.gui import *
	root = QgsProject.instance().layerTreeRoot()
	model = QgsLayerTreeModel(root)
	view = QgsLayerTreeView()
	view.setModel(model)
	view.show()

&uparrow; [Back to top](#table-of-contents)


Layers
---

__Add Vector layer__

	layer = iface.addVectorLayer("/path/to/shapefile/file.shp", "layer name you like", "ogr")

__Get Active Layer__

	layer = iface.activeLayer()

__List All Layers__

	names = [layer.name() for layer in QgsProject.instance().mapLayers().values()]
	
__Show methods__

	dir(layer)
	
__Get Features__

	for f in layer.getFeatures():
		print (f)
  
 __Get Geometry__
 
	 for f in layer.getFeatures():
	  geom = f.geometry()
	  print ('%s, %s, %f, %f' % (f['NAME'], f['USE'],
		 geom.asPoint().y(), geom.asPoint().x()))
 
__Hide a field column__

	def fieldVisibility (layer,fname):
	  setup = QgsEditorWidgetSetup('Hidden', {})
	  for i, column in enumerate(layer.fields()):
	    if column.name()==fname:
	      layer.setEditorWidgetSetup(idx, setup)
		break
	    else:
	      continue
	      
__Move geometry__      

	geom = feat.geometry()
	geom.translate(100, 100)
	feat.setGeometry(geom)

__Adding new feature__

	iface.openFeatureForm(iface.activeLayer(), QgsFeature(), False)

__Layer from WKT__

	layer = QgsVectorLayer('Polygon?crs=epsg:4326', 'Mississippi', 'memory')
	pr = layer.dataProvider()
	poly = QgsFeature()
	geom = QgsGeometry.fromWkt("POLYGON ((-88.82 34.99,-88.0934.89,-88.39 30.34,-89.57 30.18,-89.73 31,-91.63 30.99,-90.8732.37,-91.23 33.44,-90.93 34.23,-90.30 34.99,-88.82 34.99))")
	poly.setGeometry(geom)
	pr.addFeatures([poly])
	layer.updateExtents()
	QgsProject.instance().addMapLayers([layer])

&uparrow; [Back to top](#table-of-contents)

Settings
---

__Get QSettings list__

	from PyQt5.QtCore import QSettings
	qs = QSettings()

	for k in sorted(qs.allKeys()):
	    print (k)


&uparrow; [Back to top](#table-of-contents)

ToolBars
---

__Remove Toolbar__

	toolbar = iface.helpToolBar()	
	parent = toolbar.parentWidget()
	parent.removeToolBar(toolbar)

	#and add again
	
	parent.addToolBar(toolbar)

__Remove actions toolbar__

	actions = iface.attributesToolBar().actions()
	iface.attributesToolBar().clear()
	iface.attributesToolBar().addAction(actions[4])
	iface.attributesToolBar().addAction(actions[3])

&uparrow; [Back to top](#table-of-contents)

Menus
---

__Remove Menu__

	menu = iface.helpMenu()	
	menubar = menu.parentWidget()
	menubar.removeAction(menu.menuAction())

	#and add again
	menubar.addAction(menu.menuAction())


&uparrow; [Back to top](#table-of-contents)

Common PyQGIS functions
---

https://github.com/boundlessgeo/lib-qgis-commons

https://github.com/inasafe/inasafe/tree/master/safe/utilities

https://docs.qgis.org/testing/en/docs/pyqgis_developer_cookbook/index.html

https://pcjericks.github.io/py-gdalogr-cookbook/index.html

https://raw.githubusercontent.com/klakar/QGIS_resources/master/collections/Geosupportsystem/python/qgis_basemaps.py

&uparrow; [Back to top](#table-of-contents)

Sources
---

<http://docs.qgis.org/testing/en/docs/pyqgis_developer_cookbook/>

<http://qgis.org/api/>

https://qgis.org/pyqgis/3.0/

<https://stackoverflow.com/questions/tagged/qgis>

&uparrow; [Back to top](#table-of-contents)

