---
layout: page
title: "Realtime Blending of Raw Motion Captured Dance"
date: 2015-01-11 01:03
comments: true
sharing: true
footer: true
---

_[The following is an edited copy of a report for a project I did for the
University of Alberta's
[CMPUT 414 - Introduction to Multimedia Technology](https://www.cs.ualberta.ca/undergraduate-students/course-directory/introduction-multimedia-technology)
course in the Winter 2012 semester. I worked on this project with Teri Drummond,
and with additional guidance from Dr. Anup Basu and Dr. Amirhossein Firouzmanesh
(then a graduate student) of the University of Alberta. The source code for this
project is available
[on GitHub](https://github.com/mlindgren/CMPUT-414-MoCap-Project).  Work from
this project was also incorporated into the publication
"[Efficient compression of rhythmic motion using spatial segmentation and temporal blending](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6618261),"
by Dr. Amirhossein Firouzmanesh, myself, Teri Drummond, Dr. Irene Cheng, and Dr.
Anup Basu.]_

## Abstract

Fluid, complex, and precise movements come together in time to music to produce
dance. When performed by a human, dancing appears smooth and natural.  Can a
computer efficiently generate dance?  We aim to develop a means of creating and
simulating complete dance compositions by smoothly blending small segments of
motion captured dance.

First, we evaluate the previous work of Shiratori and Ikeuchi, who synthesized
robotic Japanese-style folk dance in time to music, and Sauer and Yang, who
developed a model for synthesizing Celtic line dance from a set of simple motion
primitives. We then focus our research to specific algorithms and methods, and
explore Kristine Slot's paper on motion blending methods, as well as Pejsa and
Pandzic's similar methods as applied to virtual characters directly conrolled by
the user.

Using these resources as a blueprint, we modify the
[ASF/AMC Motion Capture Viewer written by Jim McCann of Carnegie Mellon](http://www.cs.cmu.edu/~jmccann/),
adding spherical linear interpolation of quaternions and distance mapping, among
other techniques, to transition between AMC motion capture files of short dance
moves.

Finally, we review our results. Generally, our method is successful at smoothly
and naturally interpolating joint positions between motions. However, it is
limited in its ability to orient the entire figure's facing-direction.  We
assess the limitations and potential improvements of our method, and point to
possible future work on our project.

## Introduction

There are numerous potential practical applications for a robust dance
composition system.   Choreographers could use such a system to quickly and
cheaply prototype their dances before working with live dancers. The system
could also be applied to other types of choreographed action such as fighting in
movies, close-quarters battles in video games, or producing intricate dance
scenes in computer-generated cartoons. With our project, we aim to explore the
process of capturing, analyzing, and decomposing complex motion captured dance
data, with the overall goal of computationally generating realistic,
entertaining dance from a series of simple motion captured movements.

Our primary goals are as follows:

- Compare and evaluate previous work in the vein of complex motion
  synthesis.
- Begin by collecting samples of motion captured dance from the Carnegie
  Mellon University Motion Capture Database.
- Examine the Carnegie Mellon ASF/AMC Motion Capture Viewer source
  code. By modifying the  source code and adding external scripts, alter the
  viewer to display smooth dance from a handful of input motion capture files.
- Allow the creation of compositions by stringing various motions together.
- Smoothly transition between motion files, giving the appearance of natural
  dance movement.

## Previous Work

Computer generation of movement is common in the entertainment industry. It
often incorporates research into the natural capabilities and limitations of the
human body can be used for a variety of types of simulations.  Three overarching
processes of movement generation exist: (Guo, 2012)

- **Kinematics-based approaches** which depend on an artist to tune
  a body's poses, and rely on constraints to control the motion of the
  character. This is closely tied to keyframe animation.
- **Dynamics-based approaches** which use physics to calculate
  realistic motions in response to external forces upon an object or body.  This
  is dependent on constraints of joint motion, and often used in simulations.
- **Example-based approaches** drawing from motion captured databases to
  synthesize complex motions. Although the variety of possible motions is
  limited to the original set data, this method of motion generation often
  produces the most natural movement. (Guo, 2012)

Our project aims to synthesize complex dance movements using an example-based
approach.  In a similar vein to our proposed project are the papers we will
initially discuss, "Synthesis of Dance Performance" by Shiratori and Ikeuchi,
and "Music-Driven Character Animation" by Sauer and Yang. Papers which explore
our problem in more depth will also be evaluated, and their algorithms and
approaches discussed.  Kristine Slot's "Motion Blending" and Pejsa and
Pandzic's "State of the Art in Example-Based Motion Synthesis for Virtual
Characters in Interactive Applications" provide methods which we adapt and use
in our implementation. Other non-literary resources will also be used, including
the [CMU Graphics Lab Motion Capture Database](http://mocap.cs.cmu.edu/) and the
projects available at [MotionSynthesis.org](http://motionsynthesis.org/).

### Shiratori and Ikeuchi - Synthesis of Dance Performance

In their study, Shiratori and Ikeuchi review three methods for automatically
synthesizing dance movement based on music.  In brief, they describe the three
methods as follows: "The first study analyzes the relationship between motion
and musical rhythm and extracts important features for imitating human dance
motion.  The second study models modification of upper body motion based on the
speed of the played music so that the humanoid robot dances to a faster musical
speed.  The third study automatically synthesizes dance performance that
reflects both rhythm and mood of input music". (Shiratori and Ikeuchi,
2008)

Shiratori and Ikeuchi focus on three key features of input music to try to
generate representative dance sequences.  Those features are _**rhythm, speed**_
and _**mood**_.  For each of the three key features, they have developed
corresponding analysis and synthesis methods.

- **Rhythm:** Based on observation of human dance, Shiratori and Ikeuchi
  theorize that the  correspondence between movement and rhythm is captured by
  keyposes: brief pauses in movement which synchronize the "motion rhythm" with
  the musical rhythm.
- **Speed:** The authors observed that human dance motions change slightly when
  they have  to be synchronized to music played back at higher speed:
  specifically, some detail is omitted so that the performers can go through
  their movements quickly enough to stay in sync with the music.
- **Mood:** To match the performance to the mood of the music, the authors
  designed an algorithm "to synthesize new dance performance by assuming these
  relationship [sic] between motion and music rhythm mentioned in the first
  study, and the relationship between motion and music intensity" (Shiratori
  and Ikeuchi, 2008)

#### Keypose Extraction

Human movement can be decomposed into motion primitives, which "denote
fundamental elements of human motion, and ... are segmented by detecting
instances when hands and feet stop their movements" (Shiratori and Ikeuchi,
2008).  The authors establish keyposes by analyzing motion to extract “stopping
postures” and analyzing music to extract the rhythm.

The motion analysis step is "based on the speed of the performer’s hands, feet
and center of mass".  Keyposes are extracted according to the following
criteria:

1. "Dancers clearly stop their movements" during a keypose.
1. "Dancers clearly move their body parts during neighboring keyposes."

Hand and center of mass keyposes occur when the speed of the area in question
reaches a local minimum which is below some global minimum speed threshold.
Additionally, the local maximum between prospective keyposes must be greater
than some global maximum speed threshold.  Similarly for foot motions, keyposes
are extracted from periods of relative stillness between large leg motions.

The authors compared their method with previous methods for keypose extraction
using two types of traditional Japanese dance: _aizu-bandaisan_ and
_jongara-bushi_.  They concluded that their method "was much more
accurate than the previous method," and believed that the reason for the
improved accuracy was that their method "considers not only motion capture data
but musical information, while the previous method considers only motion capture
data." (Shiratori and Ikeuchi, 2008)

#### Mood-based Animation

Shiratori and Ikeuchi propose that when performers improvise dance motions based
on music that they are listening to, they "do not create these motions, but
rather combine appropriate motion segments from their knowledge database with
music as their key to perform their unique movements" (Shiratori and Ikeuchi,
2008).

The authors describe a three-step process for synthesizing dance motions
according to musical mood:

1. Motion analysis, in which the "rhythm and intensity features" of
   motion-captured dance animations are extracted. A motion database is developed
   from this step.
1. Music analysis, in which the "structure of input music sequence" is analyzed
   for "musical rhythm and intensity features." A music segment database is
   developed based on this step.
1. Synthesis, in which new motion is generated by "interpolating between motion
   segments."

Building on research by Laban, the authors develop a formula which relates "the
linear sum of approximated instantaneous momentum magnitude calculated from the
link and body directions" (Shiratori and Ikeuchi, 2008), called the
"weight effort," to the "excitement of motion."  Motion rhythm is extracted from
local minima in the weight effort, and intensity is determined according to
"momentum and forward translation."  As the details of music feature extraction
are not applicable to our project, they will not be elaborated here.

Finally, motion is synthesized by "considering both the motion and music feature
vectors." (Shiratori and Ikeuchi, 2008) First, rhythm is matched between
the music and the motion segments.  For candidate matches, "connectivity
analysis" is applied to check "if synthesized transition motion between the
neighboring motion segments looks natural." Finally, intensity matching is used
to further refine the set of candidate matches. The algorithm is described
pictorally in the following figure.

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/MoSynth.png" alt="A graphical representation of the motion synthesis algorithm"/>
<br />
<small>
Figure 1: From Shiratori et al: A graphical representation of the motion
synthesis algorithm.
</small>
</div><br />

The authors tested their methodology by synthesizing motion for Japanese
_Kansho-odori_ dance.  Regarding the results, they write: "We can easily
confirm that most of the musical rhythm is matched to the motion rhythm, and the
distributions of the intensity components are quite similar.  Some artists
working with the Japanese dance performance group _Warabi-za_ told us
that our technique was quite useful for a performance group such as theirs to
compose new Japanese dances and this would be a new method for re-use of dance
motion data." (Shiratori and Ikeuchi, 2008)

### Sauer and Yang - Music Driven Character Animation

In their study, Sauer and Yang aim to create a system which builds a dance
animation based on the attributes of a piece of music. "Using a simple script
that identifies the movements involved in the performance and their timing, the
user can easily control the animation of characters," (Sauer and Yang,
2009) creating dance animations that are found to be appealing to
viewers.

#### Animation Generation from Music

First, information is extracted from a MIDI music file. Sauer and Yang use two
techniques:

- **Beat Detection:** Using onset detection, determine the tempo
  of a music file and the length of the interbeat-interval. The result is a
  vector of the actual beats during a song.
- **Dynamics Extraction:** In a song, dynamics are the "louds" and "softs", as
  well as  transitions between them, the crescendos and decrescendos. The music
  piece is processed by an  algorithm and the 50 highest and 50 lowest values
  are stored as dynamic positions, with crescendos and decrescendos interpolated
  between them. Dynamics information is useful in matching pleasing animation to
  the nature of a song.

Next, music information is used in conjunction with user-defined motion
routines.  Five components come together to generate pleasing dance:

- **Primitive Movements:** Sauer and Yang have implemented twenty-four
  primitive movements which can be combined to create complex, interesting
  motions. These incluce heel clicks, hops, stamps, leg lifts, knee bends, and
  leg raises.
- **Script File:** "Simple text files that list Celtic
  primitives and routines that the user wants performed in the resulting
  animation." (Sauer and Yang, 2009) With the script system, Sauer and
  Yang aim to create a user friendly way to lay out dance routines to be
  performed by one or more characters, in or out of sync with each other.
- **Mappings:** In generating the animation, an algorithm maps the primitives
  defined in the script file to the beats determined through beat detection.
  Two primitives are mapped to each beat "because it provides a smoother motion
  and better transitions between primitives." Dynamics are also used to alter
  the movement of particular motions so that they correspond well with music;
  timid motions are used with softer music, while extreme motions are used with
  louder music.
- **Constraints:** Ensure that "a primitive or routine is being performed by the
  correct body part according to the rules of Celtic dance." Constraints keep
  track of foot positions and control body and movement order, producing
  realistic movement. Constraints are used in some primitive motions and all
  motion routines.
- **Routines:** A series of movement primitives are combined to create a more
  complex set of motions called a routine. Complex movements used in Celtic
  dance are pre-programmed by Sauer and Yang as routines. The user can also
  implement their own routines by specifying an order of primitives, which can
  later be called throughout their own animations.

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/AniSequence.png" alt="Sequential images displaying the different positions involved in the 'FrontClickJump' Celtic routine."/>
<br />
<small>
Figure 2: From Sauer et al: Sequential images displaying the different positions
involved in the "FrontClickJump" Celtic routine.
</small>
</div><br />

#### Evaluation of Generated Celtic Dance

Sauer and Yang evaluated the success of their algorithm by presenting a video of
their generated animation, alongside a variety of music files. The viewer
specified whether they enjoyed the animation by submitting either a "yes" or a
"no", and stating their reasons. It was found that the highest rated animation
was played with Celtic music, whereas animations played with rap or orchestral
music&mdash;"musical types that do not typically suit dancing" (Sauer and Yang,
2009)&mdash;were rated lower.

### Evaluating Shiratori and Ikeuchi, Sauer and Yang

Given that neither of these studies provided source code or links to animated
demonstrations, it is difficult to directly compare them. Instead, we will
provide some general comments on their limitations and advantages, and the
applicability of each to our project.

#### Limitations and Advantages of Shiratori and Ikeuchi

Shiratori and Ikeuchi’s methods are specifically aimed at laying the groundwork
for "a dancing entertainment robot"  (Shiratori and Ikeuchi, 2008) which
synthesizes its dance according to music.  Thus, much of their work is aimed at
generating dance which is reproducible by a humanoid robot which may have an
extremely limited range of motion, and may not be able to properly reproduce
quick movements.  These constraints may therefore be unnecessary or even
limiting given that we are working exclusively with 3D-animated performances.

Additionally, although Shiratori and Ikeuchi claim that their method is
"applicable not only to Japanese folk dance but also to different styles of
dance such as break dancing," they provide no empirical evidence of this in
their paper; it appears that the only dance styles they have actually tested
their methodology on are various forms of Japanese folk dance.  Thus, there is
some concern that the specific techniques they use to quantify motion may not be
applicable to faster and more high-energy forms of dance from which it may not
be possible to reliably extract keyposes.

In fact, Shiratori and Ikeuchi have done very little empirical testing to
determine the overall perceptual quality of their results.  While they briefly
mention some positive feedback they have received in limited testing, they later
admit that they have not done any formal evaluation of the perceptual quality of
their generated dance sequences.  Thus, it is difficult to determine from the
paper alone how successful they truly were.

One advantage to the work done by Shiratori and Ikeuchi is that it matches
fairly closely with the goals of our project, as we too would like to split
dance motion into short segments and then compose sequences from those
primitives (starting with manual composition and then potentially working up to
automatic synthesis if time allows).  That said, their primitives are based on
the movements of individual body parts, which is probably a finer level of
control than we can hope to achieve given our time constraints.

#### Limitations and Advantages of Sauer and Yang

Sauer and Yang's system is designed to "[help] users of all experience levels to
produce appealing animations based on input music of any type and primitive
dance moves and routines." Its focus on user-friendly generation of dance based
on primitives ties well with our proposed project.  Unfortunately, their paper
does not specify the methods used in creating motion primitives in the first
place. It is suggested that the Celtic dance primitives were programmed by Sauer
and Yang, rather than extracted from a pre-existing motion captured dance.
However, Sauer and Yang cite Shiratori et al.’s previous work in decomposing
motion into primitives.

Currently, Sauer and Yang’s system is limited to producing Celtic dance. In
Celtic dance, only the dancer’s legs move; their arms remain at their sides and
their torso stays upright. Dancers do not interact or dance together, but
usually execute their routines while standing in a line. Although Sauer and
Yang’s system could be expanded to handle other varieties of dance primitives,
such as those of ballroom dancing, the benefits to our project may be be
insufficient relative to the effort that would be required to expand the current
system. For example, in order to incorporate paired dancing, special attention
would need to be given to each dancer’s partner; pairs of dancers would have to
stay in sync, with each dancer also being able to move independently.

Incorporating arm, torso, and full body movements further complicates the
system.  Currently, the dance generation algorithm must only process foot and
leg movements typical of Celtic dance; it only needs to ensure that each leg is
performing one action at a time, and track which foot is in front of the other
when initializing motions relying on the current "leading foot".  Other forms of
dance involve more complicated movements, such as:

- Side-stepping foot movements in addition to forward and back steps.
- Movement of the dancer around the floor, rather than staying in place as
  typical of Celtic dance.
- Arm positioning, whether in interaction with a partner or alone.
- Full body and torso movements, such as leaning or dipping.

### Slot - Motion Blending

Krinstine Slot outlines a generic approach to blending two motions in her paper
"Motion Blending," providing methods highly applicable to our project.  Slot
outlines three components that create a linear blend between two motions:

- **Timewarping:** Determine the optimal point to begin blending two motions,
  and "performing the blend by timing the two animations so the frames being
  blended continues [sic] being good matches."
- **Alignment Curves:** "Considers the problems when blending two animations
  with  an angle larger than 180 degrees. If we use a normal linear blend, the
  blended animation will turn in an undesired direction once the angle passes
  180 degrees."
- **Constraint Matching:** Because two frames are combined by computing the
  average between them, it is possible that some constraints of the moving body
  may be violated. Constraint matches are created for each motion, and are
  considered as blending takes place.

Since the Constraint Matching aspect of this process is utilized in neither
Slot's nor our project, it will not be elaborated on in this paper.

Similarly to our project, Slot's motion blending concerns a simple character
consisting of bones connected together by joints. Each joint has a parent, with
the exception of the root joint, from which all joints branch in some
configuration.  It is important to note that "[s]ince every child moves
according to its parent, a transformation of the root will lead to a
transformation of the whole character." (Slot, 2007) Additionally, Slot
distinguishes between motion _blending_ - wherein corresponding joints'
rotations and positions are combined using weights - and motion _transition_ - a
temporary motion blend intended to "cross-fade" from one motion to another. In
motion transition, the start point of the transition from within one motion, the
end point of the transition within the second motion, and the overall length of
the transition, must all be considered.  For our project, we are primarily
concerned with motion transition.

#### Linear Blending

Slot provides two functions for achieving a linear blend of two motions by
combining the rotations and positions of corresponding joints.  "If given two
motions, <em>M<sub>i</sub></em> and <em>M<sub>j</sub></em>, which are to be
blended, one with a weight <em>w<sub>i</sub></em> and the other with a weight
<em>w<sub>j</sub></em>, then the blended position of the k'th joint of the two
motions (<em>J<sub>ik</sub></em> and <em>J<sub>jk</sub></em>)" are found by
equation 1.

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/mocap_eqn1.png" alt="Equation 1"/>
<br />
<small>
Equation 1
</small>
</div><br />

For N animations blended together, the position of the k'th joint can be found
by equation 2.

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/mocap_eqn2.png" alt="Equation 2"/>
<br />
<small>
Equation 2
</small>
</div><br />

A visual representation of this kind of linear blend can be seen in figure 3.

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/LinBlend.png" alt="A linear blend of two motions. The grey and red legs are the original motions, and the green legs are the blend between them."/>
<br />
<small>
Figure 3: From Slot: "A linear blend of two motions. The grey and red legs are
the original motions, and the green legs are the blend between them."
</small>
</div><br />

#### Timewarping

Slot notes that although the linear blending method succeeds in blending two
motions, it does not do it smartly. In order to achieve realistic, "more
plausible" motion, Slot introduces Kovar et al.'s timewarp curve method.
Ideally, two motions would be blended together at the frames where they match
closely. To accomplish this, an N &times; M distance map is constructed,
comparing the joint orientations of each possible pair of frames, where M is
"the number of frames in the source motion and N is the number of frames in the
destination motion. For each cell (i,j) in the map, the difference between the
i'th frame in the destination motion (F<sub><em>dst</em></sub>(i)) and the j'th
frame of the source motion (F<sub><em>src</em></sub>(j)) are computed." (Slot,
2007) The distance D between the i'th and j'th frames of source motion is
determined by computing the distance between each of the joint positions in
frame F<sub><em>dst</em></sub>(i) as compared to F<sub><em>src</em></sub>(j),
and then finding the weighted sum of those distances. (See equation 3.)

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/mocap_eqn3.png" alt="Equation 3"/>
<br />
<small>
Equation 3
</small>
</div><br />

Using this equation, we wish to minimize D, where p<sub><em>k,j</em></sub> is
the k'th point of the j'th frame of the source animation.

#### Testing using Distance Mapping

A visualization of Slot's timewarping method is the Distance Map, a greyscale
image representing the distance between two figures, each in a frame that must
be blended.  An _equal motions_ distance map represents a condition where the
"destination and source motions are the same" (Slot, 2007); the diagonal of the
distance mape should be zero, shown by complete black.  Similarly, a _non-equal
motions_ distance map represents the more complex condition wherein the start
and end destinations of two motions differ. This could occur, for example,
between motions moving the character at different velocities, such as a run
being blended with a walk.

Slot primarily uses the Distance Mapping technique for testing her Timewarping
method. Distance Mapping is implemented in our program, and will be explained in
more detail in the following discussion of Pejsa and Pandzic's paper.

#### Alignment Curves

Alignment curves seek to solve the problem that when "Given a blending between
two motions with a rotation, the linear blend fails once the angle between the
motions are larger than 180 degrees." (Slot, 2007) Alignment curves give each
motion a "vote" on the figure's orientation, and votes are taken into account
alongside blend weights when determining the final motion. Since all bones
branch from the root, the alignment change is only applied to the root.

Slot explains the process of calculating the alignment curve as:

> When calculating the timewarp curve, we found the angle _&theta;_ that could
> be used for transforming the destination frames local coordinate system onto
> the source frames coordinate system. This value can be used here by using this
> as the weighted orientation for deciding the final orientation. In order to
> make sure no angle is above 180 degrees, we use the alignment constrain that
> <em>|&theta;<sub>i</sub> - &theta;<sub>i</sub> -1 &le; _&pi;_|</em>. To
> fulfill this we change <em>&theta;<sub>i</sub></em> for the i'th frame on the
> timewarp curve if this criteria does not hold. When an angle passes 180
> degrees <em>&theta;<sub>i</sub></em> changes sign, hence we continue counting
> in the same sign as <em>θ<sub>i-1</sub></em>, but adding the difference of
> <em>&theta;<sub>i-1</sub></em> and <em>-&theta;<sub>i</sub></em>.

### Pejsa and Pandzic - Example-Based Motion Synthesis

In their paper "State of the Art in Example-Based Motion Synthesis for Virtual
Characters in Interactive Applications," Pejsa and Pandzic explore motion
synthesis as a solution to creating realistic, fluid movements in virtual
characters.  Their problem is unique in that it applies to user-controlled
animated characters, such as those found in games. "In offline applications all
movement scenarios are predefined and animators can plan every detail of the
characters' motion in advance. In interactive applications this is impossible,
because actions occur dynamically, depending on factors like user input and
current state of the game world." (Pejsa and Pandzic, 2010)  The majority of
game engines designed to produce realistic human movement draw from motion
captured data, yet combining these motions is still a frustrating task.
Hand-crafting motion transitions is tedious and time consuming, and many
pre-existing automatic methods such as Procedural Translation disregard the
natural constraints of the human body, creating jarring unrealistic movements.
In response to this problem, Pejsa and Pandzic propose the motion blending
algorithm _Slerp_.

#### Motion Blending with Slerp

Like Slot's approach, Pejsa and Pandzic are animating a figure composed of
bones, each with a parent.  All bones branch down in some configuration from the
root, which does not have a parent.  The root positions of each  motion can be
blended thusly:

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/mocap_eqn4.png" alt="Equation 4"/>
<br />
<small>
Equation 4
</small>
</div><br />

The position and velocities of each of these motion positions
(<em>p<sub>i</sub></em>) are weighted (<em>w<sub>i</sub></em>) and summed.

In this process, joint orientations of each position are represented as
non-singular unit quaternions, "that is, every unit quaternion _q_, along with
its antipode _-q_, maps to exactly one orientation." (Pejsa and Pandzic, 2010)
The name of Pejsa and Pandzic's algorithm, Slerp, refers to "spherical linear
interpretation" of two quaternions, and is computed as:

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/mocap_eqn5.png" alt="Equation 5"/>
<br />
<small>
Equation 5
</small>
</div><br />

...where <em>&theta; = arccos(q<sub>1</sub>q<sub>2</sub></em>) and _t_ is
the interpolation weight. (Pejsa and Pandzic, 2010)

Pejsa and Pandzic also offer methods for the more complex process of blending
_N_ motions, but since our project is concerned only with producing a "cross
fade" between two motions, they are not applicable to our implementation.

#### Distance Mapping

As mentioned in the discussion of Slot's Timewarping method, the most seamless
way of blending two motions is to start and end the blend at a point where the
animations to be combined are most similar. Pejsa and Pandzic propose the
Distance Mapping method as a solution to this problem.

To begin distance mapping, a motion graph must be constructed:

> Motion graph is a directed graph where edges correspond to segments of motion,
> while nodes serve as choice points for connecting the segments, that is each
> outgoing edge is potentially the successor to any incoming edge. An edge is
> either a portion of one of the example clips from the motion dataset or a
> transition between two such clips. A node corresponds to a single frame in one
> of the example clips. A graph walk represents a possible motion sequence. In a
> well-connected graph it is possible to generate a graph walk between any two
> nodes.

(Pejsa and Pandzic, 2010).

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/motion_graph.png" alt="An example of a motion graph for about a minute of motion data."/>
<br />
<small>
Figure 4: From Pejsa and Panzic: An example of a motion graph for about a minute
of motion data.
</small>
</div><br />

Although our program does not draw from enough possible animations to require
a motion graph, the subsequent process of distance mapping is applicable to
our implementation.

## Implementation

As a starting point, we chose to modify
[Jim McCann's ASF/AMC viewer](http://www.cs.cmu.edu/~jmccann/).  McCann's
ASF/AMC viewer was designed and tested to work with the ASF and AMC files
available on the Carnegie Mellon University Motion Capture Database.  It is
cross-platform and uses OpenGL, which we had prior experience with, to render
the ASF/AMC motions.  It also has a number of convenience classes and functions
which make it easy to work with the ASF/AMC data.  These features made it an
ideal platform on which to build our project.

### Python Script

As a first step, we created a Python script which performed extremely naïve
linear interpolation between two AMC files; this script is included in the
project source under <code>Scripts/interpolate.py</code>.  This script works by
reading two AMC files.  For each positional component of each bone specified in
the AMC files, it calculates the difference between the ending position in the
first AMC file and the starting position in the second file.  The script has no
knowledge of the skeleton specified by the ASF file; therefore, it does not
distinguish between translational and rotational components.  All positional
components are simply treated as raw numbers which are "animated" by linearly
interpolating between the starting and ending number.

The script outputs a new AMC file which contains the linear interpolation
between the two specified animations. By playing the first file, then the newly
created interpolation file, and then the second file in sequence, one arrives at
a rough transition between the two original animations.  With a simple
modification, we were able to make the ASF/AMC viewer automatically cycle
through all of the animations in its data directory, advancing to the next
animation as soon as the current animation finished.

The Python script was a convenient way to rapidly prototype our ideas without
going to the difficulty of understanding and modifying the ASF/AMC viewer's C++
code, but it proved time-consuming to use as it required the user to manually
run it on animation pairs and then properly order the three output animations by
manually ensuring that they were named in alphabetical order (the ASF/AMC viewer
loads and plays files in alphabetical order.)  More importantly, due to the
extremely simple method used to create the interpolated animation, the results
were very poor: the speed of the transition animation was always unnatural
unless the user was able to guess the correct number of frames to interpolate
based on the difference between the ending and starting positions of the two
original animations, and naïve linear interpolation of root positions and
orientations resulted in "foot-skating," sliding, and strange rotations.

### Real-time Blended Transitions in C++

Using Kristine Slot's paper as a blueprint, we moved on to implementing more
robust real-time blended transitions by directly modifying the C++ source code
of the ASF/AMC viewer.  The ASF/AMC viewer renders each frame of the motion from
a Pose object containing positional data and joint angles, as well as a pointer
to a Skeleton object, which in turn contains information about the length and
girth of each of the bones in the skeleton, and their hierarchy.

Poses are calculated on a per-frame basis from Motion objects, which contain
motion data loaded from AMC files.  Therefore, we are able to blend individual
frames by maintaining two Motion objects and two Pose objects - one of each for
each of the first and second animations.  The Motions generate the poses, and
then we can interpolate between the joint angles in each pose using the weighted
linear blending technique described in Kristine Slot's paper, which we
previously discussed.  All of this is handled by our LerpBlender class, which is
located in <code>Library/LerpBlender.cpp</code> and the associated header file
<code>Library/LerpBlender.hpp</code>.

Drawing from Pejsa and Pandzic's method, LerpBlender uses spherical linear
interpolation of quaternions to integrate joint positions. Our implementation of
Slerp assumes that, given two motions, the figure in each animation will have
the same number of bones. When looking at a pair of frames <em>from\_pose</em>
and <em>to_pose</em> to be blended, LerpBlender must interpolate a blended
orientation of each bone in <em>from\_pose</em> corresponding to its counterpart
bone in <em>to\_pose</em>.  LerpBlender loops through each pair of bones,
passing their orientation quaterion into Slerp.  The resulting interpolated bone
orientation replaces the original bone orientation in <em>from\_pose</em>. The
influence of each animation is determined by a blending coefficient which is
defined as:

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/mocap_eqn6.png" alt="Equation 6"/>
<br />
<small>
Equation 6
</small>
</div><br />

That is, the influence of the second animation grows exponentially as the frame
number of the second animation in the current blend approaches the total number
of interpolated frames.  The value is clamped to a maximum of 1, since it should
range from 0 to 1, with 0 indicating that the blend consists entirely of data
from the first animation, and 1 indicating that the blend consists entirely of
data from the second animation. The exponential growth was chosen arbitrarily,
but upon visual inspection seems to produce nicer results than calculating the
blending coefficient linearly according to the ratio of the current frame number
and the total number of interpolated frames.

After the quaternions for each bone are interpolated, the orientation is
interpolated using Slerp. Naïve blending of the root positions between
animations is extremely unreliable, as it will result in sliding and foot
skating, sometimes in the opposite direction of apparent motion (i.e. the
character may be walking in what appears to be a forwards direction, but
translating backwards.) Instead, the root position is modified according to the
velocity between frames. The velocity is calculated for each of the two motions,
and the two resulting velocities are then interpolated according to the blending
coefficient calculated previously.

### Distance Mapping

Kristine Slot's Timewarping method is roughly implemented in the DistanceMap
class, located in <code>Library/DistanceMap.cpp</code> and
<code>Library/DistanceMap.hpp</code>.  The DistanceMap class builds an
_n_ &times; _m_ map of distances between the frames in two animations, where _n_ is
the number of frames in the starting animation, and _m_ the number of frames in
the subsequent animation.  Distances between each pair of frames are calculated
using Slot's Timewarping function (see equation 3) and stored in the map.
Currently, distances are calculated on demand with the <code>getDistance</code>
function; this was done for speed as the distances between non-examined nodes
are not important when calculating the path, and calculating the distances for
_n_ &times; _m_ frames of two large animations can be very slow.  Our distance
calculation does not account for the weighting of bones.  Ideally, position
weights would be based on bone length or density, or most logically weight
(_volume_ &times; _density_), so that distances between large bones are weighted
more strongly than distances between small bones when calculating the shortest
path.

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/distancemap1.png"
alt="Figure 5: A distance map comparing two identical animations. Notice how the
diagonal is completely black, indicating that the distance between positions at
corresponding frames of the animations is zero. The image was generated in
MATLAB using data from the DistanceMap class."/>
<br />
<small>
Figure 5: A distance map comparing two identical animations. Notice how the
diagonal is completely black, indicating that the distance between positions at
corresponding frames of the animations is zero. Generated in MATLAB using data
from the DistanceMap class.
</small>
</div><br />

<div style="margin-left: auto; margin-right: auto; text-align: center;">
<img src="/images/distancemap2.png"
alt="Figure 6: A distance map comparing two differing animations.
calcShortestPath will attempt to draw a path from the start of the interpolation
to the end, passing through frames which are most similar to each other (as
indicated by darker shades of grey) without violating the slope limit.  Motion
files used to generate this data were all_data/05_01.amc and all_data/05_02.amc.
The image was generated in MATLAB using data from the DistanceMap class."/>
<br />
<small>
Figure 6: A distance map comparing two differing animations.
<code>calcShortestPath</code> will attempt to draw a path from the start of the
interpolation to the end, passing through frames which are most similar to each
other (as indicated by darker shades of grey) without violating the slope limit.
Motion files used to generate this data were <code>all_data/05_01.amc</code> and
<code>all_data/05_02.amc</code>.  The image was enerated in MATLAB using data
from the DistanceMap class.
</small>
</div><br />

As Kristine Slot's paper is primarily concerned with directly blending two
animations, she uses the Timewarping method to find an ideal starting and
finishing location for the blend, where the start may be anywhere in the first
animation and the end may be anywhere in the second (as long as rules about the
maximum number of subsequent frames to use from one animation without moving to
the next frame in the other are not violated).  This approach proved to not be
very suitable for our purposes, as we are more concerned with smoothly
transitioning between two complete animations; i.e. we want to begin our
transition near the end of the first animation and finish it near the start of
the second animation.  Therefore, we perform blending only over a fixed number
of frames relative to the animation size; currently we use one quarter of the
frames in the shorter animation.  Ideally, this value would not be a constant
defined in the code, but controllable by the user.  Our DistanceMap class, then,
starts on the frame that LerpBlender starts blending, and from there calculates
the shortest path from the start to the end, where the end cannot occur after
the maximum number of blended frames.

LerpBlender's interpolation uses DistanceMap's <code>calcShortestPath</code>
function to find the best path between frames. The calculation uses a slope
limit _L_ of 3, suggested by Slot as a sufficient constraint to prevent
unnatural movements caused by skipping between "distant" frames. Starting from
the zero'th frame of the first animation, <code>calcShortestPath</code> pushes
frame pairs onto the "shortest path" STL vector. The first element of the frame
pair is the frame of the first animation to be blended, and the second element
is the frame of the second animation.  Until the frame at which interpolation
should start is reached, only the frame in the first animation is incremented
while the frame in the second animation stays at 0. Once interpolation starts,
then pairs are chosen based on their distance, as calculated by the
<code>getDistance</code> function, as long as their location in the distance map
does not violate the slope limit.

### Blending Motionsin Series

Once two motions could be successfully blended using the above methods, the only
problem remaining was to seamlessly blend a string of motions, making the end of
each transition into to the beginning of the next. A  modification to the
<code>update</code> and <code>switch\_motion</code> functions of the BrowseMode
class, located in <code>Browser/BrowseMode.cpp</code>, and associated header
file <code>Browser/BrowseMode.hpp</code>, allows the viewer to cycle through all
AMC files in the input directory, blending smoothly between animations.

Initially, each motion file was displayed sequentially, in a location and
orientation determined by the AMC file's data alone.  The resulting display of
movement was jarring and hard to follow; when a transition between motions
occurred, the figure would instantaneously jump across the floor to a different
position and orientation.  The updated <code>switch\_motion</code> function
modifies the state of the figure in the upcoming motion according to the root
position of the current pose. The X and Z coordinates of the figure are used to
ensure that a motion begins at the same location as where the previous motion
ended. The Y coordinate&mdash;the height of the root from the ground&mdash;is
about halfway up the figure's body, and is not considered when modifying the
figure's position; alterations to the Y coordinate could cause the figure to
rise off the ground or sink below the floor during the animation.

The <code>switch\_motion</code> function updates the LerpBlender used to blend
the first and second motion. Now, when the currently displayed frame is updated
in the <code>update</code> function, and if the figure is transitioning between
motions, the blender obtains the current, interpolated pose for display.  This
process is repeated each time one motion comes to an end and another is
beginning, allowing the AMC Viewer to display a continuous stream of motion
captured files which transition and loop smoothly.

## Results

Our project's success depends on the naturalness and fluidity of the resulting
interpolations, and is difficult to measure quantitatively.  Ideally, our
outcome would be compared to the motion interpolation work of others.
Unfortunately, however, other similar projects, such as Shiratori and Ikeuchi's,
and Sauer and Yang's, did not provide source code or recordings of their
animation results.  We must instead judge the quality of our own animations as
objectively as possible.  In this section we will provide a qualitative analysis
of our results as demonstrated by four recordings of interpolated animation.

It is important to note that the low framerate of our videos is a result of the
low-quality freeware screen capture software used to record.  The live framerate
is outputted to the top-left of the screen, and is generally at about 120fps,
making the actual motion quite smooth.  Additionally, the exact times of
transition between motion files can be deduced by attending to the first row of
text information, which shows the file names being interpolated. When the file
names change, a transition has been completed.

### First Example

{% video http://files.mlindgren.ca/videos/mocap1.mp4 798 597 http://blog.mlindgren.ca/images/mocap-banner.png %}

The first recording demonstrates a simple combination of animation files,
emulating what a choreographer might put together when designing a dance. The
figure takes a few steps forward, and transitions into a twirl motion. During
the transition, one can subtly see the limbs sliding into position in
preparation for the twirl motion. After the second twirl, the figure's feet
change position.  This interpolation is a successful example of correcting the
position of large bones in a natural way; the figure's weight seems to shift to
its right leg as the left leg comes forward. However, the following transition
is less successful; the figure spins in place counter clockwise as a result of a
drastic change in orientation between motions. Although the interpolation is
correct in changing the figure's facing-direction along the shortest path, the
result is not very natural.

It is worth noting that the briefly inhuman backwards knee-bend during the final
leg raise is not a result of our interpolation, but rather an error in the
original motion capture data.

The data used for this recording were <code>05\_01.amc</code>,
<code>05\_02.amc</code>, <code>05\_03.amc</code>, and <code>05\_05.amc</code>.

### Second Example

{% video http://files.mlindgren.ca/videos/mocap2.mp4 798 597 http://blog.mlindgren.ca/images/mocap-banner.png %}

The second recording tests interpolation between drastic changes in arm
positions. The first motion ends with the figure's left arm raised, and the
second motion begins with the figure's arms lowered.  The interpolation between
motions successfully moves  the arms between positions in a natural, fluidic way
(unfortunately, the capture skips a couple of frames in this instant, making it
hard to see.  The motion can be recreated live using the files listed below).

Later, interpolation occurs between a motion ending with arms down, and a motion
beginning with arms up. This transition is again smooth and successful. Although
the joint orientation blending is natural, the overall figure orientation is
not; there is slight skating on the floor as the figure rotates in place.

The data used for this recording were <code>05\_02.amc</code>,
<code>05\_04.amc</code> and <code>05\_11.amc</code>.

### Third Example

{% video http://files.mlindgren.ca/videos/mocap3.mp4 795 596 http://blog.mlindgren.ca/images/mocap-banner.png %}

The third recording is a demonstration of how two motion files can be made to
loop continuously.  Repeating the same motion in a loop has useful applications
in dance training; a dancer could observe the motion repeatedly, and emulate it,
as the animated figure danced perpetually.

This file also demonstrates our reoccurring issue of rotation in place.  Note,
though, that the jitteriness of the final jump is again a consequence of our
poor-quality recording software; the live jump was as smooth as the previous
jumps.

The data used for this recording were <code>05\_18.amc</code> and
<code>05\_20.amc</code>.

### Fourth Example

{% video http://files.mlindgren.ca/videos/mocap4.mp4 795 596 http://blog.mlindgren.ca/images/mocap-banner.png %}

The fourth and final recording shows a figure covering a large distance during
its dance.  Interpolations seem to successfully maintain the appearance of the
figure's momentum as it travels.  In this case, the figure's overal direction of
motion is the same between all of the walk and jump/spin raw motion files, so
the final animation is quite successful. If this were not the case, as will be
discussed in the Improvements section, the results would be far less natural
given our current implementation.

The data used in this recording were <code>05\_01.amc</code>,
<code>05\_08.amc</code>, <code>05\_13.amc</code>, and <code>05\_18.amc</code>.

## Conclusions

Overall, our implementation is successful. Pejsa and Pandzic's spherical linear
interpolation of quaternions, integrating joint positions, is used in
conjunction with Slot's distance mapping, finding the shortest path between
frames for blending. With these two methods, we generally solve our primary
problem of transitioning smoothly between motions with diverse figure positions.
Because naïve blending was found to be ureliable in interpolating root
positions, we instead reposition the root between transitions by accounting for
changes in its X and Z coordinates, and recalculating based on the root's
velocity from one motion to the next.  Prior to applying this improvement, the
figure would jump sporadically between motion files. In our implementation, the
figure's positioning seems much more natural between motions.

Despite our general success, our program leaves room for improvement.
Interpolation frequently becomes unnatural when a figure's facing-direction is
drastically different at the end of one motion and the beginning of the next.
Currently, our blender resolves this discrepancy by rotating the figure in
place&mdash;a solution which could be improved, as discussed in the following
subsection.

The success of the Distance Map technique is, unfortunately, dubious. It is
difficult to test whether DistanceMap significantly improves our interpolations;
Slot's paper seems to only demonstrate the Distance Map on pairs of animations
which are already similar, leading us to believe that she (and possible the
original authors of this technique) may have only tested it on similar
animations.  Since the animations we attempted to blend were diverse and
dissimilar, it is difficult to relate our results to Slot's, when she used
motions which were much easier to blend together.

### Improvements

Presently, the DistanceMap class does not apply weights to each bone when
calculating the distance between pairs of corresponding positions. Only raw
distances are used.  Although Slot does not describe the weighting criteria for
each position, it seems intuitive that minor bones, such as fingers, should
carry less importance than larger bones, such as those representing the upper
and lower legs and arms. In a future iteration of this project, the length,
density, or most logically weight (_volume_ &times; _density_) of each bone
should be taken into account when summing distances.  Alternatively, the fact
that minor bones tend to occur in the extremities, and thus do not tend to have
many, if any, "child" bones, is worthy of consideration.  It is possible that a
sort of "automatic weighting" occurs as a consequence of this assumption; any
differences in a large bone will likely be compounded by differences in its
progressively smaller "child" bones, but differences in small bones will
themselves be of smaller magnitudes, and thus will may significantly affect
their larger "parent" bones, nor will they have many of their own "child" bones.

One of the most noticeable shortcomings of our project is its difficulty in
handling drastic changes in a figure's orientation between animations. A sample
of this problem can be seen in the third recording. After finishing a jump, the
figure can be seen slowly rotating about 180&deg; in place. This unnatural
movement is a consequence of our interpolation between an animation ending with
the figure facing one way, transitioning into an animation with the figure
facing the opposite direction.  The limitations of this problem become apparent
when trying to compose a specific motion routine.  For example, if a user has
two motion capture files, one of a walk, and the other of a run, and wants to
create a blended animation of a figure walking, transitioning to a run, our
program would be an ideal solution.  However, if the user's walking animation is
at a completely different angle of motion than the running animation, our
program would fail; rather than transitioning smoothly from a walk to a run in a
straight trajectory, the figure's direction of travel would rotate drastically
as it transitioned between animations.

One potential solution to this problem would be the use of more standardized
motion capture files.  Although useful in their diversity, the CMU Motion
Capture Database dance files are frustratingly varied in their format;
animations often have very different locations and orientations of root
positions, making interpolating between them difficult. If each motion file was
formatted to a standard starting location and orientation, a more successful and
stable interpolation method could be devised.  An more robust solution could
implement Slot's alignment curve method; an improved algorithm utilising this
technique would align an entire animation so that its starting point is oriented
similarly to the ending orientation of the previously played motion file.
Unfortunately, due to time constraints, we could not apply alignment curves to
the current iteration of this project.  An even more ambitious solution might
allow the user to choose the desired orientation of each animation, providing
more direct control of the figure's overall direction of motion.

Other improvements to our implementation deal with improving the user's control
over the resulting interpolated motion.  In some cases, a user may want to trim
frames from the start and end of their motion captured file.  Some of our test
AMC files from the CMU Motion Capture Database began with too-long pauses of the
figure preparing to perform a dance move, and ended with the figure relaxing or
stepping off to the side after finishing their movement.  Ideally, the user
would have the ability to trim these unnecessary start and end frames in a more
efficient manner than manually editing their motion capture files.

## References

- Shihui Guo. Open source program for character motion synthesis.
  [http://www.motionsynthesis.org/methodologies.html](http://www.motionsynthesis.org/methodologies.html),
  2012\.

- Tomislav Pejsa and I\.S\. Pandzic. State of the art in example-based motion
  synthesis for virtual characters in interactive applications. _Computer
  Graphics forum_, 29(1):202–226, 2010.

- D\. Sauer and Y-H. Yang. Music-driven character animation. _ACM Trans.
  Multimedia Comput. Commun._, 5(4), October 2009.

- T\. Shiratori and K. Ikeuchi. Synthesis of dance performance based on analyses
  of human motion and music. _IPSJ Online Transactions_, 1:80–93, July 2008.

- Kristine Slot. Motion blending. Copenhagen University, Department of Computer
  Science, February 2007.
