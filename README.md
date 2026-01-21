# Trivia Quiz MAS
This repository implements a MAS that plays a trivia quiz game. The implementation has been developed in DALIA, which is an extension of the [DALI](https://github.com/AAAI-DISIM-UnivAQ/DALI) logic programming language also providing a GUI.
Before proceeding in this README, notice that this is divided in the following subsections, each dicussing an important either of the design, of the implentation or of how to run the MAS on your PC.

- Project Idea and Detailed Specification.
- MAS Design according to GAIA Methodology.
- How to Install and Run the MAS.

## 1. Project Idea and Detailed Specification
Define a MAS in which there is an agent, representing a trivia quiz master, asking questions to multiple other agents which are competitors that try to win the quiz. Other than these, there are also helper agents which can all be invoked once per game from each competitor, and finally there is a quiz notary that keeps track dynamically of participants to the game, their points and who wins.

The master should be able to select random questions in a pool of pre-defined ones, to ask questions to competitors and to check if the answer is correct.

The competitors should be able to randomly answer questions they get, and are also able to use hints from external agents. In particular, they have two types of hints: one instantly gives the correct answer and the other reduces the number of possible answers from four to two. Moreover, to be able to participate, they must send a participation message to the notary as soon as they wake up.

The notary keeps track of participants to the game, of their points and determines when the game is finished, decreeing the winner. Moreover, it is responsible for skipping a question if the answers do not come within a pre-defined time interval, and to end the game if it is taking too long. Note that this agent starts the quiz as soon as a fixed minimum of participants joins the game.

Finally, note that each competitor waits a random amount time in the following two different situations.

- When answering, in order to simulate a thinking process, and the fact that answers are considered in the order they come.
- When sending the participation message, in order to introduce the possibility for later participants to still join the game, having the penalty of have been not able to answer previous questions.

## 2. MAS Design according to GAIA Methodology
GAIA metholody is followed, according to what's written on the paper written to present it and discuss all its details, and available at this [link](https://link.springer.com/article/10.1023/A:1010071910869).

### 2.1 Analysis Phase
The first phase is the analysis phase, which is in turn divided into two major sub-phases: the roles identification and the protocols identification. Both are discussed here below.

#### 2.1.1.1 Roles Identification

| **Role Schema:** | QuizNotary |
|---|---|
| **Description:** | The notary accepts participants to the game, starts the game, keeps track of participants' points and determines when the game is finished, decreeing the winner. It also skips questions or ends the game if elapsed time is higher than pre-defined thresholds. Finally, it keeps track of participants' available hints, and is their interface for them to ask any hint.|
| **Protocols and Activities:** | updateParticipants, updatePoints, startGame, checkWin, endGame, skipQuestion, askHelp |
| **Permissions:** | |
| reads | *Participants* // List of names of participants to the game |
| | *Players points* // Points associated to each participant |
| | *Available hints* // Hints still available for each participant |
| | *Game Time* // Time elapsed from the start of the game |
| | *Question Time* // Time elapsed from the start of the current question |
| | *Game State* // Current state of the game. It can be waiting, running or finished |
| changes | *Players points* // Points associated to each participant |
| | *Available hints* // Hints still available for each participant |
| | *Game State* // Current state of the game. It can be waiting, running or finished |
| **Responsibilities** | |
| **Liveness:** | |
| | QuizNotary = (updateParticipants. startGame. updatePoints. skipQuestion*. askHelp*. checkWin. endGame)^ω |
| **Safety:** | |
| | • *Player's Points* ≥ 0 |
| | • *Player's* ≤ *Winning Threshold* |
| | • *Game Duration* ≤ *Pre-defined Threshold* |
| | • *Question Duration* ≤ *Pre-defined Threshold* |

| **Role Schema:** | QuizMaster |
|---|---|
| **Description:** | The master selects random questions from a pool of pre-defined ones, asks questions to competitors and checks if the answer is correct. It also orders points updates, informs competitors when game ends and asks for a new question when all participants give a wrong answer. |
| **Protocols and Activities:** | askQuestion, checkAnswer, nextQuestion, notifyGameEnd, notifyPointsIncrease |
| **Permissions:** | |
| reads | **supplied** *Participants* // List of participants to the game |
| | **supplied** *Game State* // Current state of the game. It can be waiting, running or finished |
| | *Available Questions* // Pool of pre-defined questions |
| | *Available Answers* // Pool of pre-defined answers |
| | *Correct Answers* // Correct answer for each question |
| | *Current Answered* // Players who already tried to answer the current question |
| changes | *Available Questions* // Pool of remaining questions after a selection |
| | *Current Answered* // Players who already tried to answer the current question |
| generates | *Question* // Selected question to ask and relative possible answers|
| **Responsibilities** | |
| **Liveness:** | |
| | QuizMaster = (askQuestion. checkAnswer. (notifyCorrectAnswer. notifyPointsIncrease. | nextQuestion). notifyGameEnd)^ω |
| **Safety:** | |
| | • *Remaining Available Questions* ≥ 0 |
| | • *Correct Answer per Question* = 1 |
| | • *Available Answers per Question* = *Pre-defined Number* |
| | • *Current Answered* ≤ *Number of Participants* |

| **Role Schema:** | QuizCompetitor |
|---|---|
| **Description:** | The competitor participates in the quiz by joining the game, receiving questions, thinking about answers and giving responses. It can use hints from external helper agents and reacts to answer outcomes. Each competitor must send a participation message to the notary to join. Finally, it reacts to game outcome when game ends. |
| **Protocols and Activities:** | joinGame, answerQuestion, askForHelp, reactAnswerOutcome, reactGameOutcome |
| **Permissions:** | |
| reads | **supplied** *Available Answers* // Set of possible answers for current question |
| | *Available hints* // Hints still available to the participant |
| | **supplied** *Game State* // Current state of the game. It can be waiting, running or finished |
| changes | *Available Answers* // Possible answers modified by hints |
| generates | *Answer* // Selected answer to the question |
| **Responsibilities** | |
| **Liveness:** | |
| | QuizCompetitor = (joinGame. (askForHelp. giveAnswer. reactAnswerOutcome)*. reactGameOutcome)^ω |
| **Safety:** | |
| | • *Available Answers* ≥ 0 |
| | • *Participation to Game* = 1 |
| | • *Answer* ∈ *Set of Available Answers* |

| **Role Schema:** | Helper1 |
|---|---|
| **Description:** | This helper receives hint requests from competitors and instantly provides the correct answer. It can be invoked once per game by each competitor. |
| **Protocols and Activities:** | determineCorrectAnswer, receiveHintRequest, respondWithCorrectAnswer |
| **Permissions:** | |
| reads | *Correct Answers* // Correct answer for each question |
| | **supplied** *Game State* // Current state of the game. It can be waiting, running or finished |
| generates | *Correct Answer* // The correct answer to provide |
| **Responsibilities** | |
| **Liveness:** | |
| | Helper1 = (receiveHintRequest. determineCorrectAnswer. respondWithCorrectAnswer)^ω |
| **Safety:** | |
| | • *Correct Answer* is always available |

| **Role Schema:** | Helper2 |
|---|---|
| **Description:** | This helper receives hint requests from competitors and reduces the possible answers from four to two by eliminating two wrong answers. It can be invoked once per game by each competitor. |
| **Protocols and Activities:** | determineRemainingAnswers, receiveHintRequest, respondWithRemainingAnswers |
| **Permissions:** | |
| reads | *Correct Answers* // Correct answer for each question |
| | **supplied** *Available Answers* // Set of possible answers for current question |
| | **supplied** *Game State* // Current state of the game. It can be waiting, running or finished |
| changes | *Available Answers* // Reduced set of possible answers |
| generates | *Remaining Answers* // Two remaining answers after elimination |
| **Responsibilities** | |
| **Liveness:** | |
| | Helper2 = (receiveHintRequest. determineRemainingAnswers. respondWithRemainingAnswers)^ω |
| **Safety:** | |
| | • *Remaining Answers* > 0 |
| | • *Number of Remaining Answers* = *Pre-defined Number* |

#### 2.1.1.1 Protocols Identification

| JoinGame ||
|---|---|
| QuizParticipant | QuizNotary |
| Quiz participant informs quiz notary that he wants to be included in the next game. ||


## 3. Dalia
A containerized launcher with a GUI for multi-agent-systems written in [DALI](https://github.com/AAAI-DISIM-UnivAQ/DALI).

### Pre-Requisites:

1. install  [sicstus](https://sicstus.sics.se/)
2. clone    [DALI](https://github.com/AAAI-DISIM-UnivAQ/DALI):
    - check the compatibility table [DALI-DALIA Compatibility](docs/compatibility.md)
    - e.g. 
    ```sh
    git clone --branch 2024.10 --depth 1 https://github.com/AAAI-DISIM-UnivAQ/DALI
    ```
3. install [docker](https://docs.docker.com/engine/install/)

### Installation 

1. clone [DALIA](https://github.com/alyshmahell/dalia)
    - check the compatibility table [DALI-DALIA Compatibility](compatibility.md)
    - e.g. 
    ```sh
    git clone --branch 2025.09 --depth 1 https://github.com/alyshmahell/dalia
    ```
2. navigate into the cloned repo:
```sh
cd dalia
```

### Usage
```sh
./run --sicstus <path_to_sicstus_directory> --dali <path_to_dali_directory> --src <path_to_mas_directory> 
```

- `path_to_mas_directory` is the path to the multi agent system you've written using [DALI](https://github.com/AAAI-DISIM-UnivAQ/DALI), you can use the `example` directory found in this repository in `path_to_mas_directory` when getting started.