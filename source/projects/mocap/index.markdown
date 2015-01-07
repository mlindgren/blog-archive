---
layout: page
title: "Realtime Blending of Raw Motion Captured Dance"
date: 2015-01-06 23:00
comments: true
sharing: true
footer: true
published: false
---

_[The following is a slightly edited copy of a report for a project I did for
the University of Alberta's
[CMPUT 414 - Introduction to Multimedia Technology](https://www.cs.ualberta.ca/undergraduate-students/course-directory/introduction-multimedia-technology)
course in the Winter 2012 semester. I worked on this project with Teri Drummond,
and with additional guidance from Dr. Anup Basu and Amirhossein Firouzmanesh of
the University of Alberta. The source code for this project is
available [on GitHub](https://github.com/mlindgren/CMPUT-414-MoCap-Project).
This project was also incorporated into the publication
"[Efficient compression of rhythmic motion using spatial segmentation and temporal blending]"(http://ieeexplore.ieee.org/xpl/articleDetails.jsp?arnumber=6618261),
by Amirhossein Firouzmanesh, myself, Teri Drummond, Dr. Irene Cheng, and Dr.
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

Using these resources as a blueprint, we modify the existing Carnegie Mellon
ASF/AMC Motion Capture Viewer source code, adding spherical linear
interpolation of quaternions and distance mapping, among other techniques, to
transition between AMC motion capture files of short dance moves.

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

Computer generation of movement is common in the entertainment industry, can be
used for simulations, and explores the capabilities and natural limitations of
the human body.  Three overarching processes of movement generation exist: (Guo,
2012)

- **Kinematics-based approaches** which depend on the artist to tune
  a body's poses, and rely on constraints to control the motion of the
  character. Closely tied to keyframe animation.
- **Dynamics-based approaches** which use physics to calculate
  realistic motions in response to external forces upon an object or body.
  Dependent on constraints of joint motion, and often used in simulations.
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
the CMU Graphics Lab Motion Capture Database and the current projects of
[MotionSynthesis.org](http://motionsynthesis.org/).

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
  specifically, some detail is omitted so that theperformers can go through
  their movements quickly enough to stay in sync with the music.
- **Mood:** To match the performance to the mood of the music, the authors
  designed an algorithm "to synthesize new dance performance by assuming these
  relationship [sic] between motion and music rhythm mentioned in the first
  study, and the relationship between motion and music intensity" (Shiratori
  and Ikeuchi, 2008)

#### Keypose Extraction

Human can be decomposed into motion primitives, which "denote fundamental
elements of human motion, and ... are segmented by detecting instances when
hands and feet stop their movements" (Shiratori and Ikeuchi, 2008).  The
authors establish keyposes by analyzing motion to extract “stopping postures”
and analyzing music to extract the rhythm.

The motion analysis step is "based on the speed of the performer’s hands, feet
and center of mass".  Keyposes are extracted according to the following
criteria:

1. "Dancers clearly stop their movements" during a keypose."
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

- **Primitive Movements:** Sauer and Yang have implemented twenty
  four primitive movements which can be combined to create complex, interesting
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
music - "musical types that do not typically suit dancing" (Sauer and Yang,
2009) &mdash;were rated lower.

### Evaluating Shiratori and Ikeuchi, Sauer and Yang

Given that neither of these studies provided source code or links to animated
demonstrations, it is difficult to directly compare them. Instead, we will
provide some general comments on their limitations and advantages, and the
applicability of each to our project.

#### Limitations and Advantages of Shiratori and Ikeuchi

Shiratori and Ikeuchi’s methods are specifically aimed at laying the groundwork
for "a dancing entertainment robot"  (Shiratori and Ikeuchi, 2008) [^shira] which
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
such as those of ballroom dancing, the benefits to our project may be
disproportional to the effort required to expand the current system. For
example, in order to incorporate paired dancing, special attention would need to
be given to each dancer’s partner; pairs of dancers would have to stay in sync,
with each dancer also being able to move independently.

Incorporating arm, torso, and full body movements further complicates the
system.  Currently, the dance generation algorithm must only process foot and
leg movements typical of Celtic dance; we need only ensure that each leg is
performing one action at a time, and track which foot is in front of the other
when initializing motions relying on the current "leading foot".  Other forms of
dance involve more complicated movements, such as:

- Side-stepping foot movements in addition to forward and back steps.
- Movement of the dancer around the floor, rather than staying in place as
  typical of Celtic dance.
- Arm positioning, whether in interaction with a partner or alone.
- Full body and torso movements, when leaning or dipping.
- The difficulty of combining primitives of multiple body parts into realistic,
  pleasing dance.

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
motions are larger than 180 degrees. (Slot, 2007) Alignment curves give each
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
