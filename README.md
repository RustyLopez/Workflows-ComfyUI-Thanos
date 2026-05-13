# Workflows ComfyUI Draft - Thanos

![Preview](.github/preview.png)

This repository contains a ComfyUI workflow for video generation using the LTX model. The workflow processes video
content in clips, handling parameters such as scene dimensions (640x320), seed (322737872210006), clip durations,
offsets, and keyframes for motion capture and blending.

**Note: This is a work in progress and not yet stable.**

BTW: if you have tips on how to do this better, let me know. I'm not sure I've got the mocap pose + keyframe + leading
image sequence quite right.

My end goal is to have 4 workflows here that will work together.

You'll be able to pass in audio with multiple speakers, or video with multiple speakers, of very long lengths, and when
the workflow is done, it'll spit out a full length video, with individual clip renders stitched together as smoothly and
seamlessly as possible.

Shout out to the [LTX](https://github.com/Lightricks/LTX-2) and [Qwen](https://github.com/QwenLM/Qwen-Image) Teams.
Because they have awesome kit to work with.

I intend to use more from Qwen with their new TTS solutions. Basically, for multiple speakers, where the speakers are
overlapping one another, i'd use a
diarized transcript to find the speakers words in that space, and clone their voice from regions of the audio where only
they are speaking, and use that to
re-synth the intertwined audio for just their character, so that we can drive just their character's lip sync. Then you
can just re-blend the audio in your
video editor of choice and pick whatever coverage shot you want or use masking to lip sync the different characters in
the same frame.

## Other Ideas:

### Pose

Had another thought. Motion driving my current clip is difficult because the character is in a mech suite with broader
shoulders.

I saw some cloud only solutions that can re-proportion a pose model.

It would be nice to generate a pose for your character in a full body neutral position.

And then generate the same for your performance driving actor.

THEN, you can take the diff. And save that as a pose diff.

Then you can take in a batch or list of poses frames, iterate over all of them and apply the diff, and use that to drive
the character.

It may also be nice to be able to pass in a camera perspective change. As I've noticed that if the driving animation is
really heavily off the starting frame
camera perspective, then the models will have a lot of trouble reconciling the difference consistently when generating
key frames or motion capture video. So
ideally your diff could include camera prospective. It could detect the camera offset delta between the two poses, and
include that in the diff. Note if we
train an AI model to do this. We should get the two poses to teh same proportion first. So apply the proportion delta to
the actor image. And then the AI has
to predict the camera offset. We can train an AI on that pretty simply I think. But also it would be nice. to get a
camera widget you can drag around the pose,
to try and align it with the character you are animating. Note that you need to also be able to include the field of
view and projection matrix, offset from
camera...etc, not just the orientation.

Now that's great for a static camera. But if you are driving the performance of an adhoc, hand held, selfie cam that is
shaking all over the place, then you
could, take the AI based camera orientation estimator, but it'd run successively on every frame. The problem is you
don't have the characters updated frame to
compare it to. That's fine though you could take the character's pose, change it to the actors proportions, so the
opposite of what we did prior, and then try
to predict the projection matrix changes needed to get the character, into the actors updated camera position. And then
simply negate that, to re-project the
driving animation for the next frame of animation.

### Retry

This flow could be problematic if you want to re-generate a single clip in the middle of the series somewhere. Because
each clip depends on the clip before it for its first frame.

One possible solution is to , when generating a clip, look to see if it already has a "last_frame" generated for that
same clip. If so , by default, unless otherwise configured, the generator will use that already existing last frame as
the last keyframe, instead of generating a new one using the qwen + pose generation process. That way you should be able
to regenerate a clip, with a different seed or with some tweaked settings, but still land on an output frame that will
blend well with subsequent clips.

Of note one reason we are doing the Qwen + Pose to generate keyframes rather than just feeding forward the last frame
from a given iteration, is drift. Clips will diverge over time from the original frame, and this will stack over time if
you continue to pipe the diverged clips in to subsequent clips. If we pre-generate all the key frames from the original
reference image + pose, then we ensure we are never more than 1 degree removed from the original clip at a given
keyframe.