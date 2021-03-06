# Don't write code you aren't running
## 20th April 2021

> **Summary**: This article outlines a programming strategy where I avoid writing code paths before I have a concrete test case for them. I've had bugs before where seemingly simple code I wrote without running was slightly incorrect, and these issues can be hard to uncover in months-old code.
> 
> My new approach is to only write the code to handle input data I am actually seeing, and `assert()` on any assumptions the code makes. This way I avoid writing code paths until I have the input to actually test them with, when I get a failed assertion telling me exactly what needs to be fixed.

This is a handy coding practice that's emerged for me recently when I wanted to convert GLTF models into my own data format. When working on side projects I try to write the minimum code to see some tangible progress and build from there in straightforward steps. This way I always know what to work on next, and it minimises the time between minor victories that make the process fun and rewarding.

This strategy is heavily influenced by this fantastic article, [The Perils of Future-Coding by Sebastian Sylvan](https://www.sebastiansylvan.com/post/the-perils-of-future-coding/), which every programmer should read. Stubbornly avoiding "future-coding" is great for speeding up the development process; you see results much quicker and you avoid wasting time working on unnecessary features. But I've recently been thinking that it might be just as valuable for preventing bugs.

I've been bitten a few times by minor bugs that went unnoticed for months. Often it was an unexpected edge case, but there have been some very embarrassing moments where I've tracked down annoying bugs to sections of code that were simply never correct and had never (or rarely) been executed since they were written. 

I've recently passed the decade mark since I began programming, and in 10 years I like to think I've improved at spotting potential issues in code I write, such as potential inputs that need to be handled. But I've definitely been overzealous at times, and I've come to think that most pre-emptive code is a mistake.

As a simple example; no matter what file format you're loading a 3D model from, the faces of that model are usually specified as either triangles or quads. So when you're writing code to handle 3D models, you might reasonably write code to handle both, since it's not a particularly difficult problem.

But if you are currently only testing with triangle-based models, then if you introduce a bug in how you handle quad-based models you won't find it. And it might be days, weeks, months before that code path is hit for the first time, at which point you just get a missing model in your game with no errors reported by the loading code. You have to break out the debugger and figure out why this specific model is not loaded correctly, perhaps comparing it to other working models to see how it's different.

As an experiment, I tried to write my GLTF-loading code in a way that would avoid these insidious bugs, inspired by the advice in Sebastian Sylvan's article. I began by writing code that would just load the most simple test file I had. I used the debugger to inspect how the file data was structured and added assertions that would fail for any other type of input. e.g. the file's faces were specified as triangles, so I added an assertion that the file I was loading had triangular faces. 

Once my code could successfully load file 1, I tried loading file 2 with it. When that was working I'd move on to file 3 and so on. As long as I was thorough with my assertions, each new file would either be loaded successfully or a failed assertion would tell me exactly what part of the code needed to be changed. This was a very quick and satisfying workflow.

This code is far from complete, but now if I ever try to load a model with it which I don't handle correctly, it will immediately crash and I'll get an exact reason why, which is crucial if this happens days, weeks or months from now. The assertions will direct me to the place in the code that needs to be looked at, and most importantly I'll know my solution is working since I have a failing test case to verify it with.

As a quick example, here's a snippet from my GLTF conversion code:

```cpp
auto& accessor = model.accessors[skin.inverseBindMatrices];
assert(accessor.componentType == TINYGLTF_COMPONENT_TYPE_FLOAT);
assert(accessor.type == TINYGLTF_TYPE_MAT4);
assert(accessor.byteOffset == 0);
assert(accessor.bufferView >= 0);

auto& invBindPoseBuffView = model.bufferViews[accessor.bufferView];
assert(invBindPoseBuffView.byteStride == 0);
assert(invBindPoseBuffView.buffer >= 0);

auto& invBindPoseBuffer = model.buffers[invBindPoseBuffView.buffer];
mat4* invBindPoseMats = (mat4*)(&invBindPoseBuffer.data[invBindPoseBuffView.byteOffset]);
```

Based on my understanding of the GLTF spec, if `accessor.byteOffset` is nonzero then I believe it should be added to `invBindPoseBuffView.byteOffset` on the last line when fetching the inverse bind pose matrices from the buffer. But I'm happy to leave that assertion in there and verify that my understanding is correct when I actually have a file in which this is the case. I have yet to see one!

This is not to say that I would write model-loading code for a game engine team that crashes on the artists' machines all the time. Rather you should be getting as much test data as possible up-front rather than waiting for failure "in the wild". The important thing is to be guided by actual input data for the problem at hand.

This approach is probably obvious to some, but it's been a very satisfying discovery for me. It simplifies the problem-solving process and allows the code to grow fluidly and robustly. As Mike Acton says: ["The only purpose of any code is to transform data"](https://www.youtube.com/watch?v=rX0ItVEVjHc). Letting the data guide the code only makes sense.
