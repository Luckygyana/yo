Approach
============================
1. Clustering
2. Optimization
3. Smoothening

##### Clustering
1. The Gap Threshold (G):
This is a user-defined parameter (`gap_param` with a default value `1.3`), that acts as a threshold to decide whether two consecutive dialogues belong to the same cluster or different clusters. It's a measure of temporal distance in seconds.

2. Clustering Logic:
The algorithm sorts voiceover dialogues into groups (clusters) where each cluster consists of dialogues that follow each other with a time gap smaller than or equal to the gap threshold (G).

**Mathematically, the clustering can be defined as follows:**

- Let `D = [d0, d1, ..., dn]` be a list of dialogues, where each `di` has `start_time` and `end_time`.
- A pair of dialogues `(di, dj)` belongs to the same cluster if `dj.start_time - di.end_time <= G`, where `i < j`.
- If `dj.start_time - di.end_time > G`, then `dj` starts a new cluster.

##### Optimization
Objective Function:
The objective function to be minimized is defined as the sum of the squares of the differences between subsequent variables in a list. For a list of variables `vars` with N elements, the objective function `f(vars)` can be written as:
\[ f(vars) = \sum_{i=0}^{N-2} (vars[i+1] - vars[i])^2 \]
The goal of the optimization is to find the values of `vars` that minimize this objective function.

Equality Constraint:
The problem includes an equality constraint which is the sum of a series of fractions involving the variables `L[i]` and `vars[i]` subtracted by a constant series involving `L[i]` and `A[i]`. The equality constraint `g(vars)` must be satisfied such that:
\[ g(vars) = \sum_{i=0}^{N-1} \frac{L[i]}{vars[i]} - \sum_{i=0}^{N-1} \frac{L[i]}{A[i]} = 0 \]
This means that the sum of `L[i]/vars[i]` over all elements must equal the sum of `L[i]/A[i]` over all elements.

Inequality Constraints:
There are also inequality constraints which are expressed as a sequence of inequalities that relate consecutive variables based on the parameters `L`, `A`, and `R`. The inequality constraints `h(vars)` must be such that:
\[ h_i(vars) = \left\{
\begin{array}{ll}
      R + \left( \frac{L[0]}{vars[0]} - \frac{L[0]}{A[0]} \right) \geq 0 & \text{for } i=0 \\
      R - \left( \frac{L[0]}{vars[0]} - \frac{L[0]}{A[0]} \right) \geq 0 & \text{for } i=1 \\
      \text{Subsequent constraints build upon the previous ones} & \text{for } i \geq 2 \\
\end{array} 
\right. \]

These constraints enforce a maximum allowable deviation (`R`) between the inverse of the variables `vars[i]` and the inverse of some reference values `A[i]`. 

##### Smoothing

Follow [Duration Manipulation with Praat Software](https://www.fon.hum.uva.nl/praat/manual/Intro_8_2__Manipulation_of_duration.html)

Praat Automatically do linear smoothening. So we have to just pass which position we should give what spped. Our algorithm decides that.

Mathematically, for a given speed-up factor `s` for a segment, setting a `DurationTier` point to `1/s` means that the segment will be played back at a rate `s` times its original speed. If s > 1, the playback is faster; if s < 1, the playback is slower. By adjusting these points gradually, the change in playback speed can be made smoother, so it's less jarring to listeners. The script mathematically calculates the positions of the `DurationTier` points to create a linear transition in playback speed between adjacent audio segments.

he position of the points in the `DurationTier` is calculated based on the relaxation value (`relaxation`), speed-up factors (`s1`, `s2`, `s3`), and the total duration of the original sound (`total_duration`). The relaxation value is used to determine a transition region where the speed changes gradually.

For the first sound file:

- We calculate `relaxation` based on the current and next duration.
- `r` is calculated as the minimum of `relaxation / s1` and `relaxation / s2`, ensuring that the transition region does not exceed the designated relaxation period and remains proportional to the speed-up factors.
- Two points are then added to the `DurationTier` to define the start and end of the transition:
  - Start point: `call(duration_tier, "Add point", 0.00, 1/s1)`
  - End of transition: `call(duration_tier, "Add point", total_duration - r * s1, 1/s1)`
- A third point is added at the end of the audio (`total_duration`), at a speed (`1/speed`) that is an average of `s1` and `s2`, weighted by their relative difference:
  - `speed = s1 + s1 * (s2 - s1) / (s1 + s2)`

For the middle sound files, similar calculations are performed but with additional consideration for transitions at both the beginning and the end. 

For the last sound file:

- The starting point for the `DurationTier` is set to an intermediary speed (`1/speed`) instead of starting at `1/s2`, and the transition end uses the same logic as above.

Mathematically, we are defining a linear interpolation for the speed change over the transition region. Let's define:

- `t0`: Start time of the transition region.
- `t1`: End time of the transition region within the total_duration.
- `s(t)`: Speed function over time.

For the first sound file, we set `t0 = total_duration - r * s1` and `s(t0) = s1`. We want to find `s(t1)` at `t1 = total_duration`.

We can then define a linear interpolation of speed over time as follows:

\[ s(t) = s(t0) + \frac{(s(t1) - s(t0))}{(t1 - t0)} \cdot (t - t0) \]

Where \( t \) falls within \( t0 \) and \( t1 \), which defines a straight line between the points \( (t0, s(t0)) \) and \( (t1, s(t1)) \) on a graph of speed over time.

By setting points at \( t0 \) and \( t1 \), we're effectively dictating the slope of our line, creating a gradual transition from one speed to another over the duration of the relaxation period, avoiding sudden changes in playback speed.

Audio Outputs 
============================
## Issue 3
- Audio Qaulity Seems to be dropped 
- Tested with no smootening
- Using Ffmpeg for speed up

Samples

|     | With Smoothening | Without Smoothening |
| --- | ---           | ---       |
| Id (1656)| [Video](https://drive.google.com/file/d/1fvgfjklSiEBLWC7i_0eVHyX50NOWS4TM/view?usp=drive_link)| [Video](https://drive.google.com/file/d/1yRtvRwiO-MV1JyFRVw2pMeEyFH5T2ey6/view?usp=drive_link) |

## All Algo Comparasion


Samples

|     | Semantic Shortening | Our New Algo | Our New Algo + Semantic Shortening | ILP | ILP + Semantic Shortening
| --- | ---           | ---       | --- | ---       | --- |
| Id (1695)| [Video](https://drive.google.com/file/d/1yqQROH4AZ_CiZ5nEv7Gdfmqickw0IQEC/view?usp=drive_link)| [Video](https://drive.google.com/file/d/1jBhT8an1ywZ9ednmTpbWysdR5jfJQvG7/view?usp=drive_link) | [Video](https://drive.google.com/file/d/1xHw7R9dmMSJH2K7CsCfNMq1UZge4ocqJ/view?usp=drive_link) |


## Issue 2
- Play back rate issue: In previous algorthim we dont handle f we have less than 1x playback rate
- Handled this in preprocessing step (Add silence if output audio length is less than src audio)
- Change the Optmization Algorithm to have constraints on play back rate < 1

Samples

|     | Previous Algo | Fixed Algo | TimeStamps |
| --- | ---           | ---       | --- |
| Id (1398)| [Audio](https://drive.google.com/file/d/1lCzG5349LYWengwvaMb0PxRzZpOckmR2/view?usp=drive_link)| [Audio](https://drive.google.com/file/d/16szKWzuMN2bH023Y6XH1ErUMer4Os-Wf/view?usp=drive_link) | 0:31 - 0:35 |

## Issue 1
- Cut off Issue: Sometimes there is  10ms to 1sec cut off in dialgoue due to precision
- Handle this in post-processing step
- if cut off is there then we speed up the dialogue little bit more to fit inside that dialogue frame

Samples

|     |Baseline | Previous Algo | Fixed Algo | TimeStamps |
| --- | ---     | ---           | ---       | --- |
| Id (1411) | [Audio](https://drive.google.com/file/d/1ad0BH9tUaoviH9-4_FpmFd4xRTAurJPK/view?usp=drive_link) | [Audio](https://drive.google.com/file/d/1lzQXFa4YbpCKeessBZ3tCOkIPNQQoqOw/view?usp=drive_link)| [Audio](https://drive.google.com/file/d/1PxNuvRUM_Ht6j9uE1Zn8uGEPBLYtTMZk/view?usp=drive_link) | 2:04 - 2:15 |


## Approach 3
- Audio Smoothenting + Clsuter Based
- In this we assume for a particlular cluster start and end time will be preserved

|     |Baseline | Clustering + Realxation | Clustering + Smoothening | Clustering + Realxation + Smoothening |
| --- | ---     | ---                     | ---                       | ---             |
| Id(4410) | [Audio](/assets/audios/baseline_audio_approch3.wav) | [Audio](/assets/audios/optimal_speeds_with_naive_cluster_and_relaxation_0_5.wav) | [Audio](/assets/audios/audio_smoothening.wav)| [Audio](/assets/audios/optimal_speeds_with_naive_cluster_and_relaxation_0_5-wsmoothening.wav) | 
| Id(4514)_id | [Audio](https://drive.google.com/file/d/1je0R4Aax_A5BgIcpEqYCO-cZe_4Mav9e/view?usp=drive_link) , [Video](https://drive.google.com/file/d/1Jw0G5VwJnEFzqR2nw93R8DTnSAqrYiIO/view?usp=drive_link) | [Audio](https://drive.google.com/file/d/1WA6rOQXkjlCyDySzeeKyLDwckoBYMwAo/view?usp=drive_link) | [Audio](https://drive.google.com/file/d/14XLkRCjkVoaXS5HIlT-24rc5OmYRuUdx/view?usp=drive_link)| [Audio](https://drive.google.com/file/d/1YgGtPasSVbo520Vb6bv22_sc32_RUxM_/view?usp=drive_link) , [Video](https://drive.google.com/file/d/1i8Ma3TU1tDKMp9UlHsKC-2MfhTgYyW6w/view?usp=drive_link) |
| Id(4514)_hi | [Audio](https://drive.google.com/file/d/15Hp0qonLPCvlmwIUYXXLz5CXgD1dlB69/view?usp=drive_link) | [Audio](https://drive.google.com/file/d/1tcj52nrnJKZLeTAk2Be2x0ffIZfBQCpq/view?usp=drive_link) | [Audio](https://drive.google.com/file/d/14XLkRCjkVoaXS5HIlT-24rc5OmYRuUdx/view?usp=drive_link)| [Audio](https://drive.google.com/file/d/1OsOK6YI1N57AR6AAkv9ejvdZ2rhrWWOh/view?usp=drive_link) |

![SpeedUpOptimisation](/assets/images/plot.png)

###### Baseline
<audio controls controlsList="nodownload noplaybackrate" class='audio-smaller'><source src="/assets/audios/baseline_audio_approch3.wav"></audio>

###### Clustering + Realxation + Smoothening
<audio controls controlsList="nodownload noplaybackrate" class='audio-smaller'><source src="/assets/audios/optimal_speeds_with_naive_cluster_and_relaxation_0_5-wsmoothening.wav"></audio>

###### Clustering + Realxation
<audio controls controlsList="nodownload noplaybackrate" class='audio-smaller'><source src="/assets/audios/optimal_speeds_with_naive_cluster_and_relaxation_0_5.wav"></audio>

###### Clustering + Smoothening
<audio controls controlsList="nodownload noplaybackrate" class='audio-smaller'><source src="/assets/audios/audio_smoothening.wav"></audio>


## Approach 2

- audio smoothening + Video Smoothening
- In this we assume every dailgoue's start and end time will be preserved
- Here Gap paramter mean if less than this we will add this to previous cluster segment

###### Baseline
<video controls controlsList="nodownload" class='video-smaller'>
  <source src="/assets/videos/out_baseline.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

###### Only Audio Smoothening 
<video controls controlsList="nodownload" class='video-smaller'>
  <source src="/assets/videos/out_audio.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

###### Video Smooth + Audio Smooth (Gap Paramter 1.2)
<video controls controlsList="nodownload" class='video-smaller'>
  <source src="/assets/videos/out_video.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

###### Video Smooth + Audio Smooth (Gap Paramter 3)
<video controls controlsList="nodownload" class='video-smaller'>
  <source src="/assets/videos/output_video_2.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>



## Approach 1

- Only audio smoothening 
- In this we assume every dailgoue's start and end time will be preserved
- Gap paramter(G) mean if less than this we will add this to previous cluster segment
- Relaxation paramter(R) mean when smootehnng we take this much relaxation


#### Baseline

[Audio](/assets/audios/runid_baseline_21sec.wav)

|     | R(0.1) | R(0.2) | R(0.4) | R(0.5) | R(0.7) | R(0.9) | R(1.0) | R(1.2) | R(1.5) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| G(0.5) | [Audio](/assets/audios/final_runid_21sec__0.5_0.1.wav)| [Audio](/assets/audios/final_runid_21sec__0.5_0.2.wav)| [Audio](/assets/audios/final_runid_21sec__0.5_0.4.wav)| [Audio](/assets/audios/final_runid_21sec__0.5_0.5.wav)| [Audio](/assets/audios/final_runid_21sec__0.5_0.7.wav)| [Audio](/assets/audios/final_runid_21sec__0.5_0.9.wav)| [Audio](/assets/audios/final_runid_21sec__0.5_1.0.wav)| [Audio](/assets/audios/final_runid_21sec__0.5_1.2.wav)| [Audio](/assets/audios/final_runid_21sec__0.5_1.5.wav)|
| G(0.8) | [Audio](/assets/audios/final_runid_21sec__0.8_0.1.wav)| [Audio](/assets/audios/final_runid_21sec__0.8_0.2.wav)| [Audio](/assets/audios/final_runid_21sec__0.8_0.4.wav)| [Audio](/assets/audios/final_runid_21sec__0.8_0.5.wav)| [Audio](/assets/audios/final_runid_21sec__0.8_0.7.wav)| [Audio](/assets/audios/final_runid_21sec__0.8_0.9.wav)| [Audio](/assets/audios/final_runid_21sec__0.8_1.0.wav)| [Audio](/assets/audios/final_runid_21sec__0.8_1.2.wav)| [Audio](/assets/audios/final_runid_21sec__0.8_1.5.wav)|
| G(1.0) | [Audio](/assets/audios/final_runid_21sec__1.0_0.1.wav)| [Audio](/assets/audios/final_runid_21sec__1.0_0.2.wav)| [Audio](/assets/audios/final_runid_21sec__1.0_0.4.wav)| [Audio](/assets/audios/final_runid_21sec__1.0_0.5.wav)| [Audio](/assets/audios/final_runid_21sec__1.0_0.7.wav)| [Audio](/assets/audios/final_runid_21sec__1.0_0.9.wav)| [Audio](/assets/audios/final_runid_21sec__1.0_1.0.wav)| [Audio](/assets/audios/final_runid_21sec__1.0_1.2.wav)| [Audio](/assets/audios/final_runid_21sec__1.0_1.5.wav)|
| G(1.2) | [Audio](/assets/audios/final_runid_21sec__1.2_0.1.wav)| [Audio](/assets/audios/final_runid_21sec__1.2_0.2.wav)| [Audio](/assets/audios/final_runid_21sec__1.2_0.4.wav)| [Audio](/assets/audios/final_runid_21sec__1.2_0.5.wav)| [Audio](/assets/audios/final_runid_21sec__1.2_0.7.wav)| [Audio](/assets/audios/final_runid_21sec__1.2_0.9.wav)| [Audio](/assets/audios/final_runid_21sec__1.2_1.0.wav)| [Audio](/assets/audios/final_runid_21sec__1.2_1.2.wav)| [Audio](/assets/audios/final_runid_21sec__1.2_1.5.wav)|
| G(1.5) | [Audio](/assets/audios/final_runid_21sec__1.5_0.1.wav)| [Audio](/assets/audios/final_runid_21sec__1.5_0.2.wav)| [Audio](/assets/audios/final_runid_21sec__1.5_0.4.wav)| [Audio](/assets/audios/final_runid_21sec__1.5_0.5.wav)| [Audio](/assets/audios/final_runid_21sec__1.5_0.7.wav)| [Audio](/assets/audios/final_runid_21sec__1.5_0.9.wav)| [Audio](/assets/audios/final_runid_21sec__1.5_1.0.wav)| [Audio](/assets/audios/final_runid_21sec__1.5_1.2.wav)| [Audio](/assets/audios/final_runid_21sec__1.5_1.5.wav)|
| G(1.8) | [Audio](/assets/audios/final_runid_21sec__1.8_0.1.wav)| [Audio](/assets/audios/final_runid_21sec__1.8_0.2.wav)| [Audio](/assets/audios/final_runid_21sec__1.8_0.4.wav)| [Audio](/assets/audios/final_runid_21sec__1.8_0.5.wav)| [Audio](/assets/audios/final_runid_21sec__1.8_0.7.wav)| [Audio](/assets/audios/final_runid_21sec__1.8_0.9.wav)| [Audio](/assets/audios/final_runid_21sec__1.8_1.0.wav)| [Audio](/assets/audios/final_runid_21sec__1.8_1.2.wav)| [Audio](/assets/audios/final_runid_21sec__1.8_1.5.wav)|
| G(2.0) | [Audio](/assets/audios/final_runid_21sec__2.0_0.1.wav)| [Audio](/assets/audios/final_runid_21sec__2.0_0.2.wav)| [Audio](/assets/audios/final_runid_21sec__2.0_0.4.wav)| [Audio](/assets/audios/final_runid_21sec__2.0_0.5.wav)| [Audio](/assets/audios/final_runid_21sec__2.0_0.7.wav)| [Audio](/assets/audios/final_runid_21sec__2.0_0.9.wav)| [Audio](/assets/audios/final_runid_21sec__2.0_1.0.wav)| [Audio](/assets/audios/final_runid_21sec__2.0_1.2.wav)| [Audio](/assets/audios/final_runid_21sec__2.0_1.5.wav)|
