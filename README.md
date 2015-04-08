# Problem
We want to shift the octave of any given audio file.

# Question
1. How do you shift up or down an octave?
2. How do you tempo shift?
3. How do you select every other frame from an audio sample?

# Resources
1. [Pitch Shifting]
2. [Cycle Dirac]
3. [getsample()]

### 1. How to pitch shift according to [Pitch Shifting]
This resource explains to the reader the theory behind semitones, harmonics, and the frequency of notes which are all vital pieces of information for understanding how to shift pitch. More importantly, though, the resource describes how to shift the pitch of an audio sample by an octave. By doubling the length of the audio sample and playing it back at double the speed, you are essentially doubling the frequency of the audio and therefore shifting the audio sample's pitch by an octave. To achieve this effect using [Echonest], I can cut the tempo of the audio sample to double the frames of audio and then I can create a new [AudioData] object containing every other frame. I can do the opposite to lower the octave of an audio sample, first creating a new AudioData object containing every other frame of the sample and then cutting the tempo of the new audio sample in half.

This resource answers the question 1: how do you shift up or down an octave?

### 2. How to tempo shift according to [Cycle Dirac]
While looking for ways to shift tempo in Python, I came across a Remix-Example from [Echonest] that shows how to do what I'm trying to achieve. The following code from the [Cycle Dirac] example shows how to change the tempo of a beat:

```python
audiofile = audio.LocalAudioFile(input_filename)
bars = audiofile.analysis.bars
collect = []

    for bar in bars:
        bar_ratio = (bars.index(bar) % 4) / 2.0
        beats = bar.children()
        for beat in beats:
            beat_index = beat.local_context()[0]
            ratio = beat_index / 2.0 + 0.5
            ratio = ratio + bar_ratio # dirac can't compress by less than 0.5!
            beat_audio = beat.render()
            scaled_beat = dirac.timeScale(beat_audio.data, ratio)
            ts = audio.AudioData(ndarray=scaled_beat, shape=scaled_beat.shape, 
                            sampleRate=audiofile.sampleRate, numChannels=scaled_beat.shape[1])
            collect.append(ts)
```

This code breaks down the song into beats and changes the tempo of each beat to cause each measure to start and end slow. To manipulate the tempo of each beat, the programmer who wrote this code used the timeScale() function. The function takes two paramaters, a numpy array containing an AudioQuantum object's audio data and the ratio of how much to change the tempo (a value less than 1 increases the tempo and a value greater than 1 decreases the tempo). 

This resource answers question 2: how do you tempo shift?

### 3. How to select frames from an [AudioData] object
The [getsample()] function in the AudioData module makes it easy to retrieve frames from a song. The function takes either the index of a frame or a float representing the time in the audio that frame occurs and returns the chosen frame. The following is the source code for the [getsample()] function: 

```python
def getsample(self, index):
    if not isinstance(self.data, numpy.ndarray) and self.defer: 
        self.load() 
    if isinstance(index, int): 
        return self.data[index] 
    else: 
        return AudioData(None, self.data[index], defer=False)
```

After iterating through an [AudioData] object and using the [getsample()] function to append every other frame to a list, this new list of AudioData objects must be combined into one final object. The [assemble()] function handles this, taking a list of AudioData objects as a paramater and concatenating them into a new AudioData object. The following is the source code for [assemble()]:

```python
def assemble(audioDataList, numChannels=1, sampleRate=44100, verbose=True):
    if numChannels == 1: 
        new_shape = (sum([len(x.data) for x in audioDataList]),) 
    else: 
        new_shape = (sum([len(x.data) for x in audioDataList]),numChannels) 
    new_data = AudioData(shape=new_shape, numChannels=numChannels,  
                        sampleRate=sampleRate, defer=False, verbose=verbose) 
    for ad in audioDataList: 
        if not isinstance(ad, AudioData): 
            raise TypeError('Encountered something other than an AudioData') 
        new_data.append(ad) 
    return new_data 
```

These resources answer question 3: how do you select every other frame from an audio sample?
[Cycle Dirac]: https://github.com/echonest/remix/blob/master/tutorial/stretch/cycle_dirac.py
[Pitch Shifting]: http://www.guitarpitchshifter.com/pitchshifting.html
[getsample()]: http://echonest.github.io/remix/apidocs/echonest.remix.audio-pysrc.html#AudioData.getsample
[Echonest]: http://the.echonest.com/
[AudioData]: http://echonest.github.io/remix/apidocs/echonest.remix.audio-pysrc.html#AudioData
[assemble()]: http://echonest.github.io/remix/apidocs/echonest.remix.audio-pysrc.html#assemble
