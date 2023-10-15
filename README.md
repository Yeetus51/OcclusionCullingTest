# Unity Occlusion Culling GitHub Wiki

## Home
Edited this page today · 1 revision


## Introduction
[Occlusion culling](https://docs.unity3d.com/Manual/OcclusionCulling.html) is an optimization technique that prevents Unity from rendering GameObjects hidden from view by other objects. While Cameras already perform frustum culling to exclude Renderers outside their view, occlusion culling eliminates additional rendering operations for those obscured by nearer objects. This saves both CPU and GPU time, especially in GPU-bound projects with overdraw.

Occlusion culling works best in Scenes with well-defined, small areas separated by solid GameObjects, such as rooms connected by corridors. Unity's built-in occlusion culling requires sufficient memory and is unsuitable for projects generating Scene geometry at runtime.

The process involves baking data in the Unity Editor to determine a Camera's visibility at runtime. Unity divides the Scene into cells, generates visibility data, and merges cells to reduce data size. Cameras perform both frustum and occlusion culling when enabled. Unity utilizes the [Umbra library](https://blog.unity.com/technology/occlusion-culling-in-unity-4-3-the-basics) for occlusion culling.

In this article, I will explore various scenarios to evaluate the effectiveness of occlusion culling and its impact on performance optimization.

## Unity Culling explained

Occlusion Culling is a technique used in computer graphics to optimize rendering by removing objects that are not visible to the camera. This can significantly improve performance by reducing the number of draw calls and the amount of geometry that needs to be processed by the GPU.

## Unity Frustum Culling

Frustum culling, on the other hand, removes objects that are outside the camera's field of view, meaning they are not visible on screen. It does not consider whether an object is hidden by other objects.

![Frustrum Culling](https://docs.unity3d.com/uploads/Main/OcclusionFrustumCulling.jpg)

[Image](https://docs.unity3d.com/uploads/Main/OcclusionFrustumCulling.jpg)


## Unity Occlusion Culling

Occlusion culling removes objects that are hidden behind other objects, which helps save resources by not rendering these hidden objects. Unity uses a component called Umbra to handle occlusion culling. In the editor, Umbra processes the game scene and creates a simplified version that helps determine what should be visible during gameplay. At runtime, Umbra uses this simplified version to test if objects should be visible or not. This process is conservative, ensuring that objects are never incorrectly considered invisible, although sometimes objects might be thought to be visible when they aren't.

![Occlusion Culling](https://docs.unity3d.com/uploads/Main/OcclusionFullCulling.jpg)

[Image](https://docs.unity3d.com/uploads/Main/OcclusionFullCulling.jpg)


## Occlusion Culling Settings

To balance the trade-offs between accuracy, processing time, and data size for occlusion culling, you can adjust a few parameters. These include the [smallest hole](https://blog.unity.com/technology/occlusion-culling-in-unity-4-3-the-basics) (how detailed the input is), [smallest occluder](https://blog.unity.com/technology/occlusion-culling-in-unity-4-3-the-basics) (how detailed the output is), and [backface threshold](https://blog.unity.com/technology/occlusion-culling-in-unity-4-3-the-basics) (optimization for the amount of data). Tweaking these parameters helps achieve the best results for occlusion culling in different game scenarios.

## Testing
In order to thoroughly assess the performance of occlusion culling, we conducted a series of tests across three distinct scenarios. Each scenario was designed to represent a different type of game environment, and data was collected every 0.5 seconds using Unity. The collected data included cumulative frame count, FPS (running average of 10 frames), visible triangle count on screen, and time.


**Scenario 1 (S1):** Indoors with large walls and many objects. The camera followed a fixed path.

![Indoors_Cubes](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/Map_withCube.PNG)

**Scenario 2 (S2):** Outdoors and surrounded by objects. The camera followed a fixed path.

![Outdoors_Cubes](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/OutsideWalls.PNG)

**Scenario 3 (S3):** Outdoors with objects all in front of the camera. The camera panned out to reveal more objects while following a fixed path.

![Wall_Cubes](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/wall.PNG)

Data was collected every 0.5 seconds from Unity, which included cumulative frame count, FPS (running average of 10 frames), triangle count visible on screen, and time. Each scenario had various tests with occlusion culling enabled and disabled.

For S1 and S2, there were eight tests in total, which can be divided into four pairs. Each pair had one test with occlusion culling enabled and the other with occlusion culling disabled:

- Test 1. No objects, just occluders (to set a base reference)
- Test 2. Low-poly objects (cube with 12 triangles).

![Cube](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/cube.PNG)
- Test 3. Medium-poly objects (Unity Sphere with 768 triangles).

![Shpere](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/sphere.PNG)
- Test 4. High-poly objects (Blender monkey model with 3936 triangles).

![Shpere](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/monkey.PNG)

Scenario 3, however, only includes tests 2 and 3. The results of these tests will help us gain a better understanding of the effectiveness of occlusion culling in various game environments and situations.

## Results
**Graph Analysis**

_Reading the graph_: Each graph has one X Axis Time (sec) which stays consistent throughout the tests, 30 seconds each test. And Two Y Axis. The Y Axis on the left side refers to the FPS values, and only FPS value may be read using the left side gridlines. The Y Axis on the right side refers to the Triangle count on screen, it's important to note that the values increase from top to bottom, this is done to find correlation easier via observation. It's also important to note that the right side Y Axis in most tests is a logarithmic scale and that the numbers are to be multiplied by a thousand. This is done because the occlusion culling has drastic effects on the poly count, it helps us visualize the data.
### Scenario 1

**S1 Test 1:**
We can see that the poly count has almost no impact on the FPS. This is because of the limited number of objects. The performance bottleneck is somewhere else in the pipeline. This test was conducted to gather data on the peak performance of my machine.
![S1_T1](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/IndoorsNoModels.png)

**S1 Test 2:**
This case is rather the best-case scenario for occlusion culling. From the Graph, we can deduce that when triangle count increases, we see a more significant performance decrease when not culling. Around 27 seconds, we see that the triangle count of the not culled scene is around 200 thousand triangles while the culled scene at the same time is only rendering about 7k triangles. This leads to an FPS increase of about 300fps for the culled scene. Also, we can make a small observation, when the triangle count of culling and no culling is close or equal, the no culling fps will be higher. We notice a small example of this at t=15. This is only a couple of data points, we have to test further to draw concrete conclusions.
![S1_T2](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/IndoorsLowPolyModelsV2.png)
Thanks to the big walls in the scenes that cull the entire scene behind it, we are able to cut down the poly count at most from about 150k tris to 2k tris.

**S1 Test 3:**
In this case, we have similar results with a different magnitude. No need to discuss this since it doesn't add to our results.

**S1 Test 4:**
Similarly, there is only magnitude changes with this change. It's about the same as Test2 but this time the average fps is lower and the culling can now cull way more polygons, in the magnitudes of 50 million triangles.
![S1_T4](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/IndoorsHighPolyModels.png)

## Mini Conclusion
In this scenario where the scene consists of large walls that could hide out many objects, it seems that occlusion culling proves to be rather useful in increasing performance.

### Scenario 2

**S2 Test 3:**
In this scenario, the objects are all around us, so there are no objects that will ever be occluded by another object. This way we can have the triangle count to be the same between both the scene, culling and no culling. And we notice the behavior that we previously noticed on S1Test2 at t=15, when there is no difference in poly count between the two scenes then the scene without culling will be more performant. This is because the machine still has to calculate visibility based on the bake map, whereas without culling it will draw everything in the view screen, and thus it has the chance to skip extra calculations.
Another small observation we can notice is that the difference of FPS between the scenes is dependent on the poly count. The higher the poly count, the less the difference. Though this observation only arises by coincidence, the test was not designed to find an answer for that. But S3 should give us a better insight into it.
![S2_T4](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/OutsideMediumPolyModels.png)

**S2 Test 4:**
In this test, we notice there is no FPS difference between the scenes. There is a simple reason for this, the high poly model and its instance quantity are overwhelming the GPU so much that drawing the objects becomes the bottleneck and occlusion culling has no effect.
![S2_T4](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/OutsideHighPolyModels.png)

## Mini Conclusion
In this scenario, we notice if the scene with many objects does not have any opportunity to perform occlusion culling, baking occlusion culling data will reduce performance.

**S3 Test 3:**
This graph answers most of our questions and some new observations can be done with it. Firstly, we can confirm that the higher the poly count on screen the smaller the performance difference will be between culling and no culling. And at one point there will be no difference because the performance will be limited by the GPU. In my case, that seems to happen when there are more than 40 Million polygons on screen. I notice for the first 3 seconds of the no culling scene the fps starts around 100 fps for it to increase up to 190 fps. This behavior is newly seen, further testing would be needed to draw a definitive conclusion about it or to classify it as an outlier.
![S3_T3](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/SingleAngleMediumPolyModels.png)

**All Results:** 
In this graph we can deduce that in most cases on the Indoors scinario where Large walls occlude objects, we get more fps with occlusion culling turned on. Where as in the other open cases its not always more.

![Comparison](https://github.com/Yeetus51/OcclusionCullingTest/blob/main/Graphs/Comparing%20Avg%20Fps%20of%20All%20scenes%20with%20state%20of%20Culling.png)



## Conclusion

Occlusion Culling is a powerful optimization technique that can greatly improve performance in your Unity projects. By understanding and implementing Occlusion Culling, you can create visually stunning experiences while maintaining optimal performance levels. When used correctly, occlusion culling can significantly reduce the number of triangles rendered on-screen, increasing FPS and providing a smoother gameplay experience. However, it is essential to carefully assess when to use this technique, as it may not always result in performance gains, depending on the scene's design and object distribution.
