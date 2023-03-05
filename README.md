# STeAMed haM-ifiER (STAMMER)

If you re-arranged the frames of [Steamed Hams](https://www.youtube.com/watch?v=4jXEuIHY9ic) in the right order, could you produce the [FitnessGram Pacer Test](https://www.youtube.com/watch?v=Y82jDHRrswc)?

~video goes here~

STAMMER is a python utility for recreating any audio source with the frames of any other audio or video source.
For each frame in the audio source being recreated, STAMMER selects the frame in the source audio with the most similar spectrum, adjusts its volume as needed, and inserts that frame into the output. Despite its name, STAMMER works on any audio, not just Steamed Hams.

## Instructions (MacOS/Linux)

STAMMER requires python 3, and the following python libraries:

```
numpy
scipy
```

It also requires the command line utility `ffmpeg`.

STAMMER can be run from the command line with the command

```sh
python stammer.py <carrier track> <modulator track> <ouptut file>
```

where `<carrier track>` is the path to an audio or video file that frames will be taken from (i.e. Steamed Hams in the above example), `<modulator track>` is the path to an audio or video file that will be reconstructed using the carrier track, and `<output file>` is a path to file that will be written to. `<output file>` should have an audio or video file extension (such as `.wav`, `.mp3`, `.mp4`, etc).

Note that because of its use of os command line calls, this program does not currently work for Windows computers.

## Why and How

For several years, making increasingly outlandish edits of the skit "Steamed Hams" from *The Simpsons* has been a [trend online](https://knowyourmeme.com/memes/steamed-hams). This project was primarily inspired by an edit of "Steamed Hams" in which [the frames were sorted by pitch](https://www.youtube.com/watch?v=iWFRKZek0FI). It really is worth a listen: the choppy untintelligible vocals, steadily rising in pitch, builds tension in a strange way. Even chopped to the granularity of a frame, the music stabs and dialogue from the skit is still recognizable.

I've messed around with audio in numpy enough that this got me thinking: what else could I do with these frames? The idea of turning "Steamed Hams" into another song by manipulating its frequencies isn't completely novel: [other videos](https://www.youtube.com/watch?v=QUb3stxzTpE) have done it using a [vocoder](https://www.youtube.com/watch?v=QUb3stxzTpE). Roughly speaking vocoder divides the carrier signal (the other song) into frequency bands, and then sets the amplitude of each band to the amplitude of the modulator signal ("Steamed Hams") at that point in time. The result is an amalgamation of the two signals, where the pitch of the carrier is "sung" through the timbre of the modulator.

Like the vocoder, STAMMER divides the carrier and modulator signal into frequency bands, and calculates the amplitude of each band. The algorithm is as follows:

Let the carrier $C$ be broken into frames $C_i, 0 \leq i < n_c$ for each of its frames, and the modulator $M$ be broken into frames $M_i, 0 \leq i < n_m$ for each of the frames. We replace each frame in $M$ with the frame of $C$ where the spectrum $c_i$ of $C_i$ is most similar to the spectrum $m_i$ of $M_i$. 

The spectrum can be calculated from the frame using the Fast Fourier Transform.  Alternatively, we could also group the FFT bins into larger bins, to speed up the calculation. It may be worth looking at [Accuracy limitations and the measurement of errors in the stochastic simulation of chemically reacting systems](https://www.sciencedirect.com/science/article/pii/S0021999105003074?casa_token=-OjUKZ73eCwAAAAA:2-eLNPyv6uj7Kucqmt-ex8wKHWZQVh7A9vcFX1UKqP2QnGyovpxg4jFkxUMeAiuiuS_0be4zzQ) to consider more algorithms for comparing histograms.

We could do this too ways: with or without normalization. Without normalization, we are just replacing $d_i$ with $s_j$ that minimizes $(dist(s_j, d_i)$ for $0 \leq j < n$, where dist is the Euclidian distance. However, we get more options if we normaize the vectors, and find $s_j$ that maximizes the dot product $\frac{s_j}{|s_j|}\cdot\frac{d_i}{|d_i|}$.