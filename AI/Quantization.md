## What is quantization in machine learning?
	
Quantization is a technique for lightening the load of executing [machine learning](https://www.cloudflare.com/learning/ai/what-is-machine-learning/) and [artificial intelligence (AI)](https://www.cloudflare.com/learning/ai/what-is-artificial-intelligence/) models. It aims to reduce the memory required for [AI inference](https://www.cloudflare.com/learning/ai/inference-vs-training/). Quantization is particularly useful for [large language models (LLMs)](https://www.cloudflare.com/learning/ai/what-is-large-language-model/).

In general, quantization is a process of converting a digital signal from a highly precise format to a format that takes up less space and is somewhat less precise as a result. The goal is to make the signal smaller so that it can be processed faster. In machine learning and AI, quantization aims to make models run faster, use less computing power, or both. This ultimately allows users to run AI models on cheaper hardware while, ideally, minimally sacrificing accuracy.

To visualize how this makes AI inference take up less memory, imagine a tube of a certain width, perhaps a couple of inches, and imagine someone needs to roll lots of marbles down the tube. If rolling large marbles, only two or three can pass through a point in the tube at once. If rolling small marbles, lots more can go through at once. Therefore the use of small marbles means more marbles can go through the tube more quickly.

Quantization converts big marbles to small marbles. The information needed to calculate inferences takes up less room in memory, therefore more information can go through more quickly, and AI computation is made more efficient.

## Why use quantization?

The models used for machine learning — a type of AI — are extremely complex. They use huge amounts of memory and computational power. In fact, the rise in popularity of AI has led to power crunches: Servers running advanced machine learning models need huge amounts of electricity. In some cases, the publicly available electrical grid cannot supply it all. This has led to a number of creative solutions, from increased use of solar power to [reactivating decommissioned nuclear power plants](https://www.npr.org/2024/09/20/nx-s1-5120581/three-mile-island-nuclear-power-plant-microsoft-ai).

Quantization aims to reduce the computational load from the other side, so that the AI models themselves use less power. For developers and organizations running AI models, this can help make AI much more cost-effective, as not everyone can afford to reactivate old nuclear power plants.

## How does quantization affect precision?

To understand why quantization affects precision, imagine asking someone for directions. The person might provide a turn-by-turn list of streets, the names of each street, and the names of the streets before and after each street. Such a set of directions is highly precise but might be hard to remember. Conversely, someone might instead say something like "second left, fourth right, first left." This is much easier to remember, albeit less precise.

In AI, quantization works by reducing the number of bits used by data points. The loss of bits means some amount of precision may be lost. The result can be more errors in the output (just as a driver might misidentify the "fourth left" in the example above without the street name). However, there are different types of quantization, some of which are more precise than others.

In practice, companies or users have use cases where the quantized AI model is "good enough." The use case often does not need highly precise results. One example of such a use case is tracking social media trends and mentions without needing exact data points, focusing on overall sentiment and engagement.

## What is post-training quantization (PTQ)?

Post-training quantization (PTQ) is the application of quantization to preexisting models that have already been trained. PTQ can be applied relatively quickly to trained models. It contrasts with quantization-aware training (QAT), which takes place before a model is trained and takes quite a bit of computing power. PTQ works by converting floating-point numbers to fixed-point numbers.

## What is floating point representation?

Floating point representation is a highly precise method for representing numbers that is commonly used in machine learning and [deep learning](https://www.cloudflare.com/learning/ai/what-is-deep-learning/). Numbers stored via floating point representation take up a set amount of bits, either 16 or 32 (depending on the type of floating point representation used).

Many types of quantization reduce this to 8 bits. The result is that quantized numbers take up half or a quarter as much space in memory. Of course, with fewer bits, quantized values are not as precise as floating point numbers — just like a number with fewer decimal places (e.g., 3.14) is less precise than a number with more decimal places (e.g., 3.141592654).

## What is activation-aware weight quantization (AWQ)?

Activation-aware weight quantization (AWQ) is a technique that aims to balance efficiency improvements with precision. AWQ protects the most important weights within a model from alteration. (Weights are values that measure the relationship between items within a data set.) Think back to the directions example above. Imagine instead of "second left, fourth right, first left," the direction-giver says, "second left, right on 12th Street, then first left." This has some more precise information but is still a relatively short (and easy to remember) list of directions. AWQ works similarly, keeping some integers of a model unchanged while altering others.

[Cloudflare Workers AI](https://www.cloudflare.com/developer-platform/products/workers-ai/) supports several large language models (LLMs) that incorporate both floating point quantization and AWQ so that they are lighter on memory and use less computational power while retaining precision. To learn more, see [Cloudflare Workers AI documentation](https://developers.cloudflare.com/workers-ai/models/).