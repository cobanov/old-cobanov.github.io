# Prompting

## Prompt Weighting

Source: [AUTOMATIC1111](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Features#prompt-editing)

Using () in the prompt increases the model's attention to enclosed words, and [] decreases it. You can combine multiple modifiers:

**Cheat Sheet**

- a (word) - increase attention to word by a factor of 1.1
- a ((word)) - increase attention to word by a factor of 1.21 (= 1.1 * 1.1)
- a [word] - decrease attention to word by a factor of 1.1
- a (word:1.5) - increase attention to word by a factor of 1.5
- a (word:0.25) - decrease attention to word by a factor of 4 (= 1 / 0.25)
- a \(word\) - use literal () characters in prompt

With (), a weight can be specified like this: (text:1.4). If the weight is not specified, it is assumed to be 1.1. Specifying weight only works with () not with [].

If you want to use any of the literal ()[] characters in the prompt, use the backslash to escape them: anime_\(character\).

On 2022-09-29, a new implementation was added that supports escape characters and numerical weights. A downside of the new implementation is that the old one was not perfect and sometimes ate characters: "a (((farm))), daytime", for example, would become "a farm daytime" without the comma. This behavior is not shared by the new implementation which preserves all text correctly, and this means that your saved seeds may produce different pictures. For now, there is an option in settings to use the old implementation.

NAI uses my implementation from before 2022-09-29, except they have 1.05 as the multiplier and use {} instead of (). So the conversion applies:

- their {word} = our (word:1.05)
- their {{word}} = our (word:1.1025)
- their [word] = our (word:0.952) (0.952 = 1/1.05)
- their [[word]] = our (word:0.907) (0.907 = 1/1.05/1.05)

## Negative Prompts

Negative prompts are essential for 2.0 or greater models.

**Negative Prompt Example**
> disfigured, kitsch, ugly, oversaturated, grain, low-res, Deformed, blurry, bad anatomy, disfigured, poorly drawn face, mutation, mutated, extra limb, ugly, poorly drawn hands, missing limb, blurry, floating limbs, disconnected limbs, malformed hands, blur, out of focus, long neck, long body, ugly, disgusting, poorly drawn, childish, mutilated, mangled, old, surreal, watermark
