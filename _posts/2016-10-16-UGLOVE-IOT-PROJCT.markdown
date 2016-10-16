---
layout: post
title:  "IOT Project-UGLOVE (2)"
date:   2016-10-16 00:51:02
categories: thoughts
---

## **Case of study:**

One of my tasks was making the application understand what every hand movement mean, after decoding
the messages received from the glove, i have to think about the combination of finger movement and the corresponding effects. before begining code monkeying :p I thought that i should find a pattern to make the gesture more memorizable and intuitiv.

## **Intuitiv gesture:**

The idea was to associate finger number with an action in the app for example, by bending the index finger (finger one) you will launch the first action in the screen ,bend your index and middle fingers (action 2) and so on.
this option will give the user the ability to remmber the gesture easily, But it comes with a challenge 
which how to define different handler of one single event for example the user bend his index finger
he will choose the filers option and he bend his index again to choose the first filter in the list.


## **Solution:**

for better event handling, i created what i called MODS if you are in the main interface the user gestures will trigger only the actions of the main interface , if you are chosing a filter for your sound this is the filterMod
and so on. 
Lets remember that this is a team work so i should define the names of the mods and how to navigate back and forth between them and any team member working in task will use that name and just call a method with the name of mod that he want navigate to. this will minimize the bugs and make it more easier when integrating all the work. 

#MOD#
<br/> 
{%highlight javascript%}
/**
* BehaviorCode.cs
*/

 public static class MODS
    {
        
        public const string MAIN_INTERFACE_MODE = "main_interface_mode";

        // mode 1 in the main interface
        public const string FX_MODE = "fx_mode";
        public const string EQUALIZER_MODE = "equalizer_mode";
        public const string LOOP_MODE = "loop_mode";

        //mode 2 in the fxMode
        public const string FX_DELAY_MODE = "fx_delay_mode";
        public const string FX_TREMOLO_MODE = "fx_tremolo";
        public const string FX_BIQUAD_MODE = "fx_biquad";
        public const string FX_HEIGHPASS_MODE = "fx_hieghpass";
        public const string FX_LOWPASS_MODE = "fx_lowPass";
        public const string FX_ECHO_MODE = "fx_echo";

        //mode 3 delay

        public const string FX_DELAY_SAMPLES = "fx_delay_samples";
        public const string FX_DELAY_FEEDBACK = "fx_delay_feedback";
        public const string FX_DELAY_VOLUME = "fx_delay_volume";
        public const string FX_DELAY_BLEND = "fx_delay_blend";

        //mode 4 biquad
        public const string FX_BIQUAD_RESONNANCE = "fx_biquad_resonnance";
        public const string FX_BIQUAD_CUTOFF = "fx_biquad_cutoff";

        //mode 5 LowPass Mod
        public const string FX_LOWPASS_RESONNANCE = "fx_lowPass_resonnance";
        public const string FX_LOWPASS_CUTOFF = "fx_lowPass_cutoff";

    }
    {%endhighlight%}  




<br/> 
{%highlight javascript%}
/**
* BehaviorCode.cs
*/
public class BehaviourCode
{


    private static string mode="";
    private static string previous = "";
    private static bool handOpened = true;
    private static bool leftHandOpened = true;


    public static void handOpenedTest(DecoderGuide decoder)
    {
        
        if (decoder.Hands[0].H4 == 0&& decoder.Hands[0].H3 == 0&& decoder.Hands[0].H2 == 0&& decoder.Hands[0].H1 == 0) 
            handOpened = true;
        if (decoder.Hands[1].H4 == 0 && decoder.Hands[1].H3 == 0 && decoder.Hands[1].H2 == 0 && decoder.Hands[1].H1 == 0)
            leftHandOpened = true;
    }

    public static void mainInterfaceBehaviour(DecoderGuide decoder, Button fxButton, Button equalizerButton, Button loopButton)
    {
        if (mode.Equals(MODS.MAIN_INTERFACE_MODE))
        {
            if (testFingersCondition(decoder.Hands[0], new int[] { 0, 1, 2 }, new int[] { 3 }))//H4 should be 0 else should be 1
                loopButton.onClick.Invoke();
            if (testFingersCondition(decoder.Hands[0], new int[] { 0, 1, 3 }, new int[] { 2 }))//H3 should be 0 else should be 1
                equalizerButton.onClick.Invoke();
            if (decoder.Hands[0].H4 == 1&& decoder.Hands[0].H3 == 0&& decoder.Hands[0].H2 == 0&& decoder.Hands[0].H1 == 0)//H2
            {
                fxButton.onClick.Invoke();
                BehaviourCode.setModes(BehaviourCode.MODS.FX_MODE);
            }

        }
    }

    public static void fxPanelBehaviour(DecoderGuide decoder,Button delayButton,Button lowPassButton)
    {
        if (mode.Equals(MODS.FX_MODE))
        {
            
            if (decoder.Hands[0].H4 == 1 && decoder.Hands[0].H3 == 0 && decoder.Hands[0].H2 == 0 && decoder.Hands[0].H1 == 0 && handOpened)
            {
                Debug.Log("DELAY");
                delayButton.onClick.Invoke();  
                //Delay
            }
           
            if (decoder.Hands[0].H4 == 1 && decoder.Hands[0].H3 == 1 && decoder.Hands[0].H2 == 0 && decoder.Hands[0].H1 == 0 && handOpened)
            {
                Debug.Log("LOWWPass");
                lowPassButton.onClick.Invoke();
            }
            if ((testFingersCondition(decoder.Hands[0], new int[] { 0, 1, 2 }, new int[] { 3 })) &&
                   (testFingersCondition(decoder.Hands[1], new int[] { 1, 2 }, new int[] { 3, 0 })))
            {
                //Biquad
            }
            if ((testFingersCondition(decoder.Hands[0], new int[] { 0, 1 }, new int[] { 3, 2 })) &&
                  (testFingersCondition(decoder.Hands[1], new int[] { 1, 2 }, new int[] { 3, 0 })))
            {
                //Echo
            }
            if ((testFingersCondition(decoder.Hands[0], new int[] { 0 }, new int[] { 3, 2, 1 })) &&
                 (testFingersCondition(decoder.Hands[1], new int[] { 1, 2, 0 }, new int[] { 3 })))
            {
                //Tremlo
            }
            if ((testFingersCondition(decoder.Hands[0], new int[] { 0 }, new int[] { 3, 2, 1 })) &&
                 (testFingersCondition(decoder.Hands[1], new int[] { 1, 2 }, new int[] { 3, 0 })))
            {
                //heighPass
            }
        }

        /*  if ((testFingersCondition(decoder.Hands[0], new int[] { 0 }, new int[] { 3, 2, 1 })) &&
               (testFingersCondition(decoder.Hands[1], new int[] { 1, 2 }, new int[] { 3, 0 })))
          {
              //equilizer
          }*/


    }

{%endhighlight%}  










