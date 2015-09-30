 <i class="icon-file"></i> 如何找到控制一个节点的所有动画曲线
-------------

#### Solution A:
This will get keys on the node, and dig through the anim layers to find them as well.
```python
import maya.cmds as mc
curves = mc.ls(mc.listHistory('joint1'), type = 'animCurve')
```

#### Solution B:
MAnimUtil docs.
This is great, since it does all the heavy lifting for you: Find all the attrs on the node that have incoming connections, and then for those connections, find the animCurves controling them.
```python
import maya.OpenMaya as om
import maya.OpenMayaAnim as oma

# Get a MDagPath for the given node name:
node = 'myNodeName'
selList = om.MSelectionList()
selList.add(node)
mDagPath = om.MDagPath()
selList.getDagPath(0, mDagPath)

# Find all the animated attrs:
mPlugArray = om.MPlugArray()
oma.MAnimUtil.findAnimatedPlugs(mDagPath, mPlugArray)
animCurves = []

# Find the curves ultimately connected to the attrs:
for i in range(mPlugArray.length()):
    mObjArray = om.MObjectArray()
    oma.MAnimUtil.findAnimation(mPlugArray[i], mObjArray)
    for j in range(mObjArray.length()):
        depNodeFunc = om.MFnDependencyNode(mObjArray[j])
        animCurves.append(depNodeFunc.name())
  
# See what we found:      
for ac in sorted(animCurves):
    print ac
```

#### Solution C:
The below example will print a list of all the animation curves on the selected objects, in animation layers or not.
```python
# Python code
import maya.OpenMaya as om

animCurves = []
# Create a MSelectionList with our selected items:
selList = om.MSelectionList()
om.MGlobal.getActiveSelectionList(selList)

# Create a selection list iterator for what we picked:
mItSelectionList = om.MItSelectionList(selList)
while not mItSelectionList.isDone():
    mObject = om.MObject()  # The current object
    mItSelectionList.getDependNode(mObject)
    # Create a dependency graph iterator for our current object:
    mItDependencyGraph = om.MItDependencyGraph(mObject,
                                               om.MItDependencyGraph.kUpstream,
                                               om.MItDependencyGraph.kPlugLevel)
    while not mItDependencyGraph.isDone():
        currentItem = mItDependencyGraph.currentItem()
        dependNodeFunc = om.MFnDependencyNode(currentItem)
        # See if the current item is an animCurve:
        if currentItem.hasFn(om.MFn.kAnimCurve):
            name = dependNodeFunc.name()
            animCurves.append(name)
        mItDependencyGraph.next()
    mItSelectionList.next()

# See what we found:
for ac in sorted(animCurves):
    print ac
```

#### Solution D:
Well, honestly, this really isn't a solution, but I wanted to put it in here anyway:
```python
import maya.cmds as mc
node = "pCube1"
curves = mc.findKeyframe(node, curve=True, attribute='rotateX')
```
> **Tip:** The findKeyframe command will return back the animCurve for a specific attribute, based on the current animLayer.