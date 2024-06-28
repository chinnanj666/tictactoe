<script>
  import Field from "./Field.svelte";

  let winCombos = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];

  let arr = [];
  let usersTurn = true;
  let message = "";
  let difficulty = 'Easy'; // Default difficulty level

  const winner = (perc) => {
    if (arr.length >= 5) {
      let arr2 = arr.filter((val, i) => i % 2 == perc);

      winCombos.forEach((array) => {
        if (array.every((element) => arr2.includes(element))) {
          if (perc == 0) {
            message = "You Won!";
            const game = document.querySelector(".game-frame");
            array.forEach((arr, index) => {
              game.childNodes[arr].style.backgroundColor = "#58D68D";
            });
          } else {
            message = "PC Won!";
            const game = document.querySelector(".game-frame");
            array.forEach((arr, index) => {
              game.childNodes[arr].style.backgroundColor = "#EC7063";
            });
          }
        }
      });
    }
  };

  const userMove = (e) => {
    if (e.target.firstChild.innerHTML == "" && !message && usersTurn) {
      e.target.firstChild.innerHTML = "X";
      arr.push(parseInt(e.target.getAttribute("number")));
      winner(0);
      usersTurn = false;
      if (arr.length != 9) {
        pcMove();
      }
      if (arr.length == 9 && !message) {
        message = "Game Tie!";
      }
    }
  };

  const pcMove = () => {
    if (!message) {
      setTimeout(() => {
        if (difficulty === 'Easy') {
          pcMoveEasy();
        } else if (difficulty === 'Medium') {
          pcMoveMedium();
        } else if (difficulty === 'Hard') {
          pcMoveHard();
        }
      }, 500);
    }
  };
  const pcMoveEasy = () => {
    let num;
    do {
      num = Math.floor(Math.random() * 9);
    } while (arr.includes(num));

    const game = document.querySelector(".game-frame");
    game.childNodes[num].firstChild.innerHTML = "O";

    arr.push(num);
    usersTurn = true;
    winner(1);
  };

  const pcMoveMedium = () => {
    let num;
    let arr2 = arr.filter((val, i) => i % 2 == 0);
    let arr3 = arr.filter((val, i) => i % 2 == 1);

    for (let array of winCombos) {
      if (
        array.some((element) => arr2.includes(element)) &&
        !array.some((element) => arr3.includes(element))
      ) {
        const findCommon = (a, b) => a.filter((n) => b.indexOf(n) !== -1);
        const common = findCommon(array, arr2);

        const findMissing = (a, b) => a.filter((n) => b.indexOf(n) == -1);
        const missing = findMissing(array, common);

        if (common.length == 2 && !arr.includes(missing[0])) {
          num = missing[0];
          break;
        }
      }
    }

    if (num === undefined) {
      do {
        num = Math.floor(Math.random() * 9);
      } while (arr.includes(num));
    }

    const game = document.querySelector(".game-frame");
    game.childNodes[num].firstChild.innerHTML = "O";

    arr.push(num);
    usersTurn = true;
    winner(1);
  };

  const pcMoveHard = () => {
    let bestScore = -Infinity;
    let move;
    for (let i = 0; i < 9; i++) {
      if (!arr.includes(i)) {
        arr.push(i);
        let score = minimax(arr, 0, false);
        arr.pop();
        if (score > bestScore) {
          bestScore = score;
          move = i;
        }
      }
    }

    const game = document.querySelector(".game-frame");
    game.childNodes[move].firstChild.innerHTML = "O";

    arr.push(move);
    usersTurn = true;
    winner(1);
  };

  const minimax = (newBoard, depth, isMaximizing) => {
    let result = checkWinner(newBoard);
    if (result !== null) {
      if (result === "O") return 10 - depth;
      else if (result === "X") return depth - 10;
      else return 0;
    }

    if (isMaximizing) {
      let bestScore = -Infinity;
      for (let i = 0; i < 9; i++) {
        if (!newBoard.includes(i)) {
          newBoard.push(i);
          let score = minimax(newBoard, depth + 1, false);
          newBoard.pop();
          bestScore = Math.max(score, bestScore);
        }
      }
      return bestScore;
    } else {
      let bestScore = Infinity;
      for (let i = 0; i < 9; i++) {
        if (!newBoard.includes(i)) {
          newBoard.push(i);
          let score = minimax(newBoard, depth + 1, true);
          newBoard.pop();
          bestScore = Math.min(score, bestScore);
        }
      }
      return bestScore;
    }
  };

  const checkWinner = (board) => {
    let arr2 = board.filter((val, i) => i % 2 == 0);
    let arr3 = board.filter((val, i) => i % 2 == 1);

    for (let combo of winCombos) {
      if (combo.every((element) => arr2.includes(element))) {
        return "X";
      } else if (combo.every((element) => arr3.includes(element))) {
        return "O";
      }
    }
    return board.length === 9 ? "tie" : null;
  };

  const reset = () => {
    message = "";
    arr = [];
    usersTurn = true;

    const game = document.querySelector(".game-frame").childNodes;

    game.forEach((el) => {
      el.style.backgroundColor = "";
      el.firstChild.innerHTML = "";
    });
  };
</script>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 100%;
    margin: 0 auto;
  }

  .game-frame {
    max-width: 500px;
    min-height: 500px;
    margin: 0 auto;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(3, 1fr);
    gap: 10px;
  }

  .title {
    color: white;
    margin-top: auto;
    margin-bottom: 1px;
    font-size: 46px;
  }

  .button {
    color: white;
    background-color: #333;
    padding: 10px 20px;
    border-radius: 5px;
    margin-top: 20px;
    font-size: 20px;
    cursor: pointer;
    border: none;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    transition: background-color 0.3s, transform 0.3s;
  }

  .button:hover {
    background-color: #555;
    transform: translateY(-2px);
  }

  .button:active {
    background-color: #777;
    transform: translateY(2px);
  }

  .difficulty-selector {
    color: white;
    margin-bottom: 25px;
  }
</style>

<main>
  <h1 class="title">{!message ? 'Tic Tac Toe' : message}</h1>
  <div class="difficulty-selector">
    <label for="difficulty">Difficulty: </label>
    <select id="difficulty" class="button" bind:value={difficulty} disabled={!!message}>
      <option value="Easy">Easy</option>
      <option value="Medium">Medium</option>
      <option value="Hard">Hard</option>
    </select>
  </div>
  <div class="game-frame">
    {#each Array(9) as field, index}
      <Field onClick={userMove} number={index} class="button"/>
    {/each}
  </div>
  <button class="button" on:click={reset}>{!message ? 'Reset' : 'Play Again'}</button>
</main>