// Simple Random Number Generator
class Random {
    static int seed;
    
    // Initialize with a default seed
    function void init() {
        let seed = 12345;
        return;
    }
    
    // Set custom seed
    function void setSeed(int newSeed) {
        let seed = newSeed;
        return;
    }
    
    // Generate next random number
    function int rand() {
        let seed = seed * 1309 + 13849;
        if (seed < 0) {
            let seed = ~seed;  // Make positive if negative
        }
        return seed;
    }
    
    // Generate random number in range [0, max)
    function int randRange(int max) {
        var int rand;
        let rand = Random.rand();
        return rand - ((rand / max) * max);  // Modulo operation
    }
}

class MemoryGame {
    field Array cards;        // Array to store card values
    field Array revealed;     // Array to track revealed cards
    field Array matched;      // Array to track matched cards
    field int cursorX, cursorY;  // Cursor position
    field int moves;         // Number of moves made
    field int firstCard;     // Index of first revealed card
    field int secondCard;    // Index of second revealed card
    field boolean gameOver;  // Game state

    // Constructor
    constructor MemoryGame new() {
        do Random.init();  // Initialize random number generator
        let cards = Array.new(16);      // Create 16 cards
        let revealed = Array.new(16);    // Track revealed status
        let matched = Array.new(16);     // Track matched status
        let cursorX = 0;
        let cursorY = 0;
        let moves = 0;
        let firstCard = -1;
        let secondCard = -1;
        let gameOver = false;
        
        do initializeCards();
        return this;
    }

    // Initialize cards with pairs of numbers 1-8
    method void initializeCards() {
        var Array numbers;
        var int i, rand, temp;
        
        let numbers = Array.new(16);
        // Fill array with pairs of numbers 1-8
        let i = 0;
        while (i < 8) {
            let numbers[i] = i + 1;
            let numbers[i + 8] = i + 1;
            let i = i + 1;
        }
        
        // Shuffle array using Fisher-Yates algorithm
        let i = 15;
        while (i > 0) {
            let rand = Random.randRange(i + 1);
            let temp = numbers[i];
            let numbers[i] = numbers[rand];
            let numbers[rand] = temp;
            let i = i - 1;
        }
        
        let cards = numbers;
        return;
    }

    // Handle keyboard input
    method void handleInput(char key) {
        if (key = 87) { do moveUp(); }    // W
        if (key = 83) { do moveDown(); }  // S
        if (key = 65) { do moveLeft(); }  // A
        if (key = 68) { do moveRight(); } // D
        if (key = 32) { do flipCard(); }  // Space
        return;
    }

    // Move cursor up
    method void moveUp() {
        if (cursorY > 0) {
            let cursorY = cursorY - 1;
        }
        return;
    }

    // Move cursor down
    method void moveDown() {
        if (cursorY < 3) {
            let cursorY = cursorY + 1;
        }
        return;
    }

    // Move cursor left
    method void moveLeft() {
        if (cursorX > 0) {
            let cursorX = cursorX - 1;
        }
        return;
    }

    // Move cursor right
    method void moveRight() {
        if (cursorX < 3) {
            let cursorX = cursorX + 1;
        }
        return;
    }

    // Flip card at cursor position
    method void flipCard() {
        var int index;
        let index = (cursorY * 4) + cursorX;
        
        // Check if card is already matched or revealed
        if (matched[index] | revealed[index]) {
            return;
        }
        
        if (firstCard = -1) {
            let firstCard = index;
            let revealed[index] = true;
        } else {
            if (secondCard = -1) {
                let secondCard = index;
                let revealed[index] = true;
                let moves = moves + 1;
                do checkMatch();
            }
        }
        return;
    }

    // Check if revealed cards match
    method void checkMatch() {
        if (cards[firstCard] = cards[secondCard]) {
            do Output.printString("Yea!");
            let matched[firstCard] = true;
            let matched[secondCard] = true;
            // Check if game is over
            if (isGameOver()) {
                let gameOver = true;
                do showScore();
                do restart();
            }
        } else {
            // Hide cards after a delay
            do Sys.wait(1000);
            let revealed[firstCard] = false;
            let revealed[secondCard] = false;
        }
        let firstCard = -1;
        let secondCard = -1;
        return;
    }

    // Check if all cards are matched
    method boolean isGameOver() {
        var int i;
        let i = 0;
        while (i < 16) {
            if (~matched[i]) {
                return false;
            }
            let i = i + 1;
        }
        return true;
    }

    // Show score
    method void showScore() {
        do Output.printString("Game Over! Moves: ");
        do Output.printInt(moves);
        return;
    }

    // Restart game
    method void restart() {
        do initializeCards();
        let moves = 0;
        let firstCard = -1;
        let secondCard = -1;
        let gameOver = false;
        let cursorX = 0;
        let cursorY = 0;
        
        // Reset revealed and matched arrays
        var int i;
        let i = 0;
        while (i < 16) {
            let revealed[i] = false;
            let matched[i] = false;
            let i = i + 1;
        }
        return;
    }

    // Dispose of memory
    method void dispose() {
        do cards.dispose();
        do revealed.dispose();
        do matched.dispose();
        do Memory.deAlloc(this);
        return;
    }

    // Draw game board
    method void draw() {
        var int i, x, y, value;
        let i = 0;
        while (i < 16) {
            let x = (i - ((i / 4) * 4)) * 50 + 10;  // Calculate x position
            let y = (i / 4) * 50 + 10;              // Calculate y position
            
            // Draw card
            if (matched[i]) {
                // Draw empty space for matched cards
                do Screen.setColor(false);
                do Screen.drawRectangle(x, y, x + 40, y + 40);
            } else {
                do Screen.setColor(true);
                do Screen.drawRectangle(x, y, x + 40, y + 40);
                
                if (revealed[i]) {
                    // Draw card number
                    let value = cards[i];
                    do Output.moveCursor(y/11, x/8);
                    do Output.printInt(value);
                }
            }
            let i = i + 1;
        }
        
        // Draw cursor
        do Screen.setColor(true);
        do Screen.drawRectangle(cursorX * 50 + 8, cursorY * 50 + 8,
                              cursorX * 50 + 12, cursorY * 50 + 12);
        return;
    }
}

// Main game class
class Main {
    function void main() {
        var MemoryGame game;
        var char key;
        
        let game = MemoryGame.new();
        
        while (~(key = 81)) {  // Q to quit
            let key = Keyboard.keyPressed();
            do game.handleInput(key);
            do game.draw();
            do Sys.wait(50);
        }
        
        do game.dispose();
        return;
    }
}