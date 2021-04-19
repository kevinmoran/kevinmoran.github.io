# Don't write code you aren't running
### 16th April 2021

> **Summary**: I've endeavoured to avoid writing code paths that I can't run in the debugger while writing them. Instead I write a lot of asserts to catch inputs that require new behaviour so that I can write it with an actual test case. Previously I'd happily write what looks correct, but I've been bitten in the past by bugs that went unnoticed for months because they were in untested code paths.

This is a handy coding practice that's crystallised for me recently, whilst trying to convert GLTF models into my own data format. When it comes to side projects, I try to write the minimum code to see some tangible progress and build from there in straightforward steps. This way I always know what to work on next, and it minimises the time between minor victories that make the process fun and rewarding.

So when I was trying to load a 3D model exported to a GLTF file, I decided to apply this piecewise strategy to the task. Instead of writing code to convert any arbitrary GLTF file, my job was just to handle **_the_** GLTF file I had to hand.

This made things a lot simpler! The code was tiny and straightforward, instead of branching on different input types I just had a lot of asserts; for example, assert that there is only one "scene" in the file, assert that that scene contains only one mesh, etc. This is a rather extreme extrapolation of the Data-Oriented Design idea of "solving the right problem", by reducing the "problem" you're solving to a single concrete test case.

This definitely helps speed up the development process; you spend less time coding by ignoring extraneous or theoretical codepaths, and you see results much quicker! As I thought about it though I came to think that this is perhaps even more valuable as a bug-prevention practice.

I've been bitten a few times in my side project by minor bugs that went unnoticed for months. Often it was an unexpected edge case, but there have been some very embarrassing moments where I've tracked down annoying bugs to sections of code that was simply never correct. 

I've recently hit the decade mark since I began programming, and in 10 years I like to think I've got better at spotting potential issues in code I write, such as potential inputs that need to be handled. But I've definitely been overzealous at times, and I've come to think that pre-emptively handling differences in input is a mistake.

As a simple example; no matter what file format you're loading a 3D model from, the faces of that model are usually specified as either triangles or quads. So when you're writing code to handle 3D models, you might reasonably write code to handle both. But if you are currently only testing your code with triangle-based models, then if there is a bug in the code for quad-based models you will not find it! And it might be days, weeks, months before that code sees a file with quads in it, at which point you just get a missing model in your game with no errors reported by the loading code. You have to break out the debugger and figure out why this specific model is not loaded correctly, perhaps comparing it to other working models to see how it's different.

Better instead, I say, to only write the code that will load **_exactly the data you have in front of you_**, i.e. one specific 3D model. As you're writing this code, you're using the debugger to inspect what data is actually there. If its faces are specified as triangles, you add an assertion to that effect and write the code to handle triangular faces. Then (days, weeks or months later), when you or someone else tries to load a model with quad faces, the code immediately crashes and you get an exact reason why. You can now write the code to load a 3D model with quad faces, and crucially you'll know if that code is working since you have a test case that is currently failing.

As Mike Acton says, all code does is transform data. Don't write code for data you don't have! Assert early and often.