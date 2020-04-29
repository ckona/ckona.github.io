---
layout: post
title:  Reactチュートリアルをやってみて
date:   2020-04-29 23:21:03 +0900
---

## ■ やってみたこと

___

- [Reactチュートリアル](https://ja.reactjs.org/tutorial/tutorial.html)
  - 五目並べを作りつながらReactを学んでいくチュートリアル
- [チュートリアルの最終結果](https://codepen.io/gaearon/pen/gWWZgR?editors=0010) に少し手を加えた。
  - 各componentをファイルに分割
  - React Hooks を使うように変更
  - Jest, Enzymeでテストを実装
- コードはここに置いてあります <https://github.com/ckona/tic-tac-toe-with-react>

## ■ チュートリアルの最終結果

___

```jsx
function Square(props) {
  return (
    <button className="square" onClick={props.onClick}>
      {props.value}
    </button>
  );
}

class Board extends React.Component {
  renderSquare(i) {
    return (
      <Square
        value={this.props.squares[i]}
        onClick={() => this.props.onClick(i)}
      />
    );
  }

  render() {
    return (
      <div>
        <div className="board-row">
          {this.renderSquare(0)}
          {this.renderSquare(1)}
          {this.renderSquare(2)}
        </div>
        <div className="board-row">
          {this.renderSquare(3)}
          {this.renderSquare(4)}
          {this.renderSquare(5)}
        </div>
        <div className="board-row">
          {this.renderSquare(6)}
          {this.renderSquare(7)}
          {this.renderSquare(8)}
        </div>
      </div>
    );
  }
}

class Game extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      history: [
        {
          squares: Array(9).fill(null)
        }
      ],
      stepNumber: 0,
      xIsNext: true
    };
  }

  handleClick(i) {
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = this.state.xIsNext ? "X" : "O";
    this.setState({
      history: history.concat([
        {
          squares: squares
        }
      ]),
      stepNumber: history.length,
      xIsNext: !this.state.xIsNext
    });
  }

  jumpTo(step) {
    this.setState({
      stepNumber: step,
      xIsNext: (step % 2) === 0
    });
  }

  render() {
    const history = this.state.history;
    const current = history[this.state.stepNumber];
    const winner = calculateWinner(current.squares);

    const moves = history.map((step, move) => {
      const desc = move ?
        'Go to move #' + move :
        'Go to game start';
      return (
        <li key={move}>
          <button onClick={() => this.jumpTo(move)}>{desc}</button>
        </li>
      );
    });

    let status;
    if (winner) {
      status = "Winner: " + winner;
    } else {
      status = "Next player: " + (this.state.xIsNext ? "X" : "O");
    }

    return (
      <div className="game">
        <div className="game-board">
          <Board
            squares={current.squares}
            onClick={i => this.handleClick(i)}
          />
        </div>
        <div className="game-info">
          <div>{status}</div>
          <ol>{moves}</ol>
        </div>
      </div>
    );
  }
}

// ========================================

ReactDOM.render(<Game />, document.getElementById("root"));

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

## ■ 最終的にファイル分割して、React Hooks を使うようにしたVer

___

- `/src/index.js` を `Game` コンポーネントの呼び出しをするだけに。
- `/src/components/` ディレクトリを切り、そこに各コンポーネントを移動する。
- classコンポーネントをやめて、React Hooks を使ってみる
  - thisが無くなるので見やすくなりますね。

### index.js

```jsx
import React from 'react';
import ReactDOM from 'react-dom';

import './index.css';
import Game from './components/Game';

ReactDOM.render(
  <Game />,
  document.getElementById('root')
);
```

### Square

```jsx
import React from 'react';

const Square = props => (
  <button className="square" onClick={props.onClick}>
    {props.value}
  </button>
);

export default Square;
```

### Board

```jsx
import React from 'react';
import Square from './Square';

const Board = props => {
  const renderSquare = i => (
    <Square
      value={props.squares[i]}
      onClick={() => props.onClick(i)}
    />
  );

  return (
    <div>
      <div className="board-row">
        {renderSquare(0)}
        {renderSquare(1)}
        {renderSquare(2)}
      </div>
      <div className="board-row">
        {renderSquare(3)}
        {renderSquare(4)}
        {renderSquare(5)}
      </div>
      <div className="board-row">
        {renderSquare(6)}
        {renderSquare(7)}
        {renderSquare(8)}
      </div>
    </div>
  );
};

export default Board;
```

### Game

```jsx
import React, { useState } from 'react';
import Board from './Board';

const Game = () => {
  const [history, setHistory] = useState([{squares: Array(9).fill(null)}]);
  const [stepNumber, setStepNumber] = useState(0);
  const [xIsNext, setXIsNext] = useState(true);

  const handleClick = i => {
    const _history = history.slice(0, stepNumber + 1);
    const current = _history[_history.length - 1];
    const squares = current.squares.slice();
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    squares[i] = xIsNext ? 'X' : 'O';
    setHistory(_history.concat([{squares: squares,}]));
    setStepNumber(_history.length);
    setXIsNext(!xIsNext);
  };

  const jumpTo = step => {
    setStepNumber(step);
    setXIsNext((step % 2) === 0);
  };

  const current = history[stepNumber];

  const moves = history.map((step, move) => {
    const desc = move ?
          'Go to move #' + move :
          'Go to game start';
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{desc}</button>
      </li>
    );
  });

  const status = () => {
    const winner = calculateWinner(current.squares);
    if (winner) {
      return 'Winner: ' + winner;
    } else {
      return 'Next player: ' + (xIsNext ? 'X' : 'O');
    }
  };

  const calculateWinner = squares => {
    const lines = [
      [0, 1, 2],
      [3, 4, 5],
      [6, 7, 8],
      [0, 3, 6],
      [1, 4, 7],
      [2, 5, 8],
      [0, 4, 8],
      [2, 4, 6]
    ];
    for (let i = 0; i < lines.length; i++) {
      const [a, b, c] = lines[i];
      if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
        return squares[a];
      }
    }
    return null;
  };

  return (
    <div className="game">
      <div className="game-board">
        <Board
          squares={current.squares}
          onClick={(i) => handleClick(i)}
        />
      </div>
      <div className="game-info">
        <div>{status()}</div>
        <ol>{moves}</ol>
      </div>
    </div>
  );
};

export default Game;
```

## ■ Jest, Enzyme でテストを実装

___

### 準備

#### 必要なライブラリ

- jest
- enzyme
- enzyme-adapter-react-16
- react-test-renderer

**最新のVerを下記コマンドで調べて、yarn addで追加する**

e.g.

```
$ npm info jest
$ yarn add --dev jest@25.4.0
```

**最終的な package.json**

```json
  "devDependencies": {
    "enzyme": "3.11.0",
    "enzyme-adapter-react-16": "1.15.2",
    "jest": "25.4.0",
    "react-test-renderer": "16.13.1"
  }
```

#### テストコードを書くまで



### 実際のテストコード
