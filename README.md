# PiCube
PiCube is intended to try to answer the question: Can the digits of Pi solve a Rubik's cube.
The digits are mapped to moves so that, e.g., a 3 makes an F2 move on the cube.

Digits of pi (or any other digits) are given in a Digits File. The program supports
several formats. The file extension is used to infer the format:
    .Txt - Text. 1 digit per byte.
    .B8  - 2 digits per byte.
    .B16 - 4 digits in every 2 bytes.
    .B24 - 7 digits in every 3 bytes.
    .B32 - 9 digits in every 4 bytes.
    .B40 - 12 digits in every 5 bytes.
Of these, the most useful are .txt (because digits of Pi are available online in that format)
and .B40, because it provides the most efficient compression without going to extreme measures.

A good source for digits files is: https://pi2e.ch/blog/2017/03/10/pi-digits-download/.

PiCube can convert between digits file formats with:
    PiCube --convert pi.txt pi.b40

PiCube takes a config file that specifies the type of Rubik's Cube Puzzle.
A sample config, Config_3x3.txt, is provided for the traditional 3x3 Rubik's cube.

The most basic command specifies a config file and a digits file:

    PiCube --c Config_3x3.txt --d pi.b40

You can specify more than one config file. For example, you may want to use one file
to specify the layout of the cube and another file to specify the way that digits are
mapped to moves:

    PiCube --c 3x3_Layout.txt Map.txt --d pi.b40
    
You can specify multiple digits files. For example, one file may contain the some
number of digits of pi, the second file may contain a continuation of the sequence, etc.

    PiCube --c Config_3x3.txt --d pi_00000.b40 pi_00001.b40 pi_000002.b40

You can also specify to increment the name of the digits file to derive the next name:

    PiCube --c Config_3x3.txt --d pi_00000.b40 --i

This will use all the digits in pi_00000.b40, then increment to pi_00001.b40, etc.
It will keep going until a solution is found, or it runs out of files to read, or
the numeric portion of the file name cannot be incremented any more (e.g. it reaches
pi_99999.txt).

You can specify a save file name with -s to save progress when the program terminates.
By default, the save file is progress.txt:

   PiCube --c Config_3x3.txt -d Pi_00000.b40 --i --s progress.txt

Save files are written to be used to resume where it left off. You can run:

   PiCube --c Config_3x3.txt -d Pi_00000.b40 --s progress.txt

   Then resume with:

   PiCube --c Config_3x3.txt progress.txt -d Pi_00001.b40 --s progress.txt


Here are the entries that a config file can contain:

# Text
    Lines starting with # are ignored.
// Text
    Comments from // to the end of the line are ignored.

Surfaces = 54
    Defines the number of surfaces on a cube puzzle. E.g., a 3x3 Rubik's cube could be laid out like:
    
               +--+--+--+
               | 0| 1| 2|
               +--+--+--+
               | 3| 4| 5|
               +--+--+--+
               | 6| 7| 8|
      +--+--+--+--+--+--+--+--+--+
      |09|10|11|18|19|20|27|28|29|
      +--+--+--+--+--+--+--+--+--+
      |12|13|14|21|22|23|30|31|32|
      +--+--+--+--+--+--+--+--+--+
      |15|16|17|24|25|26|33|34|35|
      +--+--+--+--+--+--+--+--+--+
               |36|37|38|
               +--+--+--+
               |39|40|41|
               +--+--+--+
               |42|43|44|
               +--+--+--+
               |45|46|47|
               +--+--+--+
               |48|49|50|
               +--+--+--+
               |51|52|53|
               +--+--+--+

StartState = 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53
    This sample start state specifies an array, where each surface N is in position N.
    The state of the cube puzzle to start with.
    A progress file will contain a StartState. Using the progress file as a config file
    will let the program resume from the state that it ended at.

StopState = 2,5,8,1,4,7,0,3,6,53,52,51,50,49,48,47,46,45,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,42,39,36,43,40,37,44,41,38,35,34,33,32,31,30,29,28,27
    The StopState is a state where the cube is solved, or any state you want it to stop in.
    For example, you may want the cube to stop when you reach a certain pattern instead of a "solved" state.
    You can have any number of stop states. The StopStates in the sample Config_3x3 show the 24
    different ways that a solved cube can be oriented.

Move F = 0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,47,46,45,18,19,20,21,22,23,15,16,17,27,28,29,30,31,32,24,25,26,42,39,36,43,40,37,44,41,38,35,34,33,48,49,50,51,52,53
    A Move is followed by a name and then a list of surfaces. The move shows how the surfaces are
    moved. E.g., this 'F' move shows that surfaces 0-14 are unaffected, but that the surface in position 47
    is moved to position 15. You can have any number of moves, and it's okay to specify moves that
    you don't actually use in the current configuration.

MoveMap
    Declares a new MoveMap, which associates digits with moves.
    You can specify more than one MoveMap with rules to switch from one MoveMap to the next.

0 = M
    A move specifies a series of one or more digits and a move that results from those digits.
    In this case, 0 = U means that when a 0 is encountered, move 'U' will be applied to the cube.
    The digits can be up to 5 digits long, but all moves in a MoveMap must use the same number of digits.
    A move may also be:
        NoOp - Do nothing.
        Repeat - Repeat the last move.
        NextMoveMap - Switch to the next move map.

MovesPerMap = 12
    This entry is optional. If specified, it gives the number of moves to make on one move map before
    switching to the next move map. MovesPerMap = 12 will switch MoveMaps every 12 moves.



A progress file uses the following items to allow the program to resume where it left off:

MovesMade = 4406156
    The number of moves made so far.

MoveMapID = 0
    The move map to start with.

MovesOnThisMap = 4406156
    The number of moves that have been made on the current MoveMap.

LastMoveIndex = 10
    The last move made.
