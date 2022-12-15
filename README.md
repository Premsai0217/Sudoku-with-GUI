# Sudoku-with-GUI
import time

# Import and initialize the pygame library
import pygame

pygame.init()

# Create the font that will be used
font = pygame.font.SysFont('Monospace', 32)


class PydokuCell:

    def _init_(self, number):
        self.number = number
        self.editable = number == 0
        self.valid = True
        self.selected = False
        self.underConsideration = False


class PydokuGUI:

    # colors that are used
    COLOR_BLACK = (0, 0, 0)
    COLOR_RED = (255, 0, 0)
    COLOR_GREY = (105, 105, 105)
    COLOR_WHITE = (255, 255, 255)
    COLOR_GREEN = (0, 255, 0)
    COLOR_BLUE = (0, 180, 255)

    # Static width calculation based on 9 cells, 6 normal lines and 2 thicker lines
    WIDTH = 9 * 39 + 6 * 1 + 2 * 3
    # Helper array to have access to the center of the cells
    cellCenters = [20, 60, 100, 142, 182, 222, 264, 304, 344]

    def _init_(self):
        self.screen = pygame.display.set_mode((self.WIDTH, self.WIDTH))
        pygame.display.set_caption('Pydoku GUI')
        self.selectedCell = None
        self.board = self.getBoard()

    def getBoard(self):
        return [
            [
                PydokuCell(0), PydokuCell(0), PydokuCell(9),
                PydokuCell(2), PydokuCell(1), PydokuCell(8),
                PydokuCell(0), PydokuCell(0), PydokuCell(0)
            ],
            [
                PydokuCell(1), PydokuCell(7), PydokuCell(0),
                PydokuCell(0), PydokuCell(9), PydokuCell(6),
                PydokuCell(8), PydokuCell(0), PydokuCell(0)
            ],
            [
                PydokuCell(0), PydokuCell(4), PydokuCell(0),
                PydokuCell(0), PydokuCell(5), PydokuCell(0),
                PydokuCell(0), PydokuCell(0), PydokuCell(6)
            ],
            [
                PydokuCell(4), PydokuCell(5), PydokuCell(1),
                PydokuCell(0), PydokuCell(6), PydokuCell(0),
                PydokuCell(3), PydokuCell(7), PydokuCell(0)
            ],
            [
                PydokuCell(0), PydokuCell(0), PydokuCell(0),
                PydokuCell(0), PydokuCell(0), PydokuCell(5),
                PydokuCell(0), PydokuCell(0), PydokuCell(9)
            ],
            [
                PydokuCell(9), PydokuCell(0), PydokuCell(2),
                PydokuCell(3), PydokuCell(7), PydokuCell(0),
                PydokuCell(5), PydokuCell(0), PydokuCell(0)
            ],
            [
                PydokuCell(6), PydokuCell(0), PydokuCell(0),
                PydokuCell(5), PydokuCell(0), PydokuCell(1),
                PydokuCell(0), PydokuCell(0), PydokuCell(0)
            ],
            [
                PydokuCell(0), PydokuCell(0), PydokuCell(0),
                PydokuCell(0), PydokuCell(4), PydokuCell(9),
                PydokuCell(2), PydokuCell(5), PydokuCell(7)
            ],
            [
                PydokuCell(0), PydokuCell(9), PydokuCell(4),
                PydokuCell(8), PydokuCell(0), PydokuCell(0),
                PydokuCell(0), PydokuCell(1), PydokuCell(3)
            ]
        ]

    def drawBoard(self):
        self.validateBoard()
        self.drawLines()
        self.drawNumbers()
        pygame.display.flip()

    def drawLines(self):
        # Fill the background
        self.screen.fill(self.COLOR_WHITE)

        # Draw the lines for the Sudoku board
        x = 40
        for i in range(1, 10):
            self.drawLine((x, 0), (x, self.WIDTH), self.COLOR_BLACK)
            self.drawLine((0, x), (self.WIDTH, x), self.COLOR_BLACK)

            # If its the third line we want to make it appear thicker,
            # so we draw three consecutive lines
            if i % 3 == 0:
                x += 1
                self.drawLine((x, 0), (x, self.WIDTH), self.COLOR_BLACK)
                self.drawLine((0, x), (self.WIDTH, x), self.COLOR_BLACK)
                x += 1
                self.drawLine((x, 0), (x, self.WIDTH), self.COLOR_BLACK)
                self.drawLine((0, x), (self.WIDTH, x), self.COLOR_BLACK)

            x += 40

    def drawLine(self, startPoint, endPoint, color):
        pygame.draw.line(self.screen, color, startPoint, endPoint)

    def validateBoard(self):
        for rowIdx in range(9):
            for columnIdx in range(9):
                cell = self.board[rowIdx][columnIdx]

                if cell.number == 0:
                    cell.valid = True
                else:
                    cell.valid = self.isValid(cell.number, rowIdx, columnIdx)

    def drawNumbers(self):
        # Go through the board and draw the numbers into the cells
        for rowIdx in range(len(self.board)):
            for columnIdx in range(len(self.board[rowIdx])):

                cell = self.board[rowIdx][columnIdx]

                if cell.number != 0:
                    color = self.COLOR_GREY if cell.editable else self.COLOR_BLACK
                    self.drawNumber(cell.number, rowIdx, columnIdx, color)

                if cell.underConsideration:
                    self.colorCellBorder(rowIdx, columnIdx, self.COLOR_GREEN)
                elif not cell.valid:
                    self.colorCellBorder(rowIdx, columnIdx, self.COLOR_RED)
                elif cell.selected:
                    self.colorCellBorder(rowIdx, columnIdx, self.COLOR_BLUE)

    def drawNumber(self, number, rowIdx, columnIdx, color):
        text = font.render(str(number),
                           True,
                           color,
                           self.COLOR_WHITE)

        textRect = text.get_rect()

        rowCenter = self.cellCenters[rowIdx]
        columnCenter = self.cellCenters[columnIdx]

        textRect.center = (columnCenter, rowCenter)

        self.screen.blit(text, textRect)

    def colorCellBorder(self, rowIdx, columnIdx, color):
        rowCenter = self.cellCenters[rowIdx]
        columnCenter = self.cellCenters[columnIdx]

        self.drawLine((columnCenter - 20, rowCenter - 20),
                      (columnCenter + 20, rowCenter - 20),
                      color)

        self.drawLine((columnCenter - 20, rowCenter + 20),
                      (columnCenter + 20, rowCenter + 20),
                      color)

        self.drawLine((columnCenter - 20, rowCenter - 20),
                      (columnCenter - 20, rowCenter + 20),
                      color)

        self.drawLine((columnCenter + 20, rowCenter - 20),
                      (columnCenter + 20, rowCenter + 20),
                      color)

    def setSelectedCell(self, mouseClickPosition):
        clickedColumnIdx = self.getCellFromCoord(mouseClickPosition[0])
        clickedRowIdx = self.getCellFromCoord(mouseClickPosition[1])

        if self.selectedCell is not None:
            rowIdx, columnIdx = self.selectedCell
            self.board[rowIdx][columnIdx].selected = False

        self.selectedCell = (clickedRowIdx, clickedColumnIdx)
        self.board[clickedRowIdx][clickedColumnIdx].selected = True

    def getCellFromCoord(self, coordinate):
        for idx, cellCenter in enumerate(self.cellCenters):
            if cellCenter - 20 < coordinate and coordinate < cellCenter + 20:
                return idx

        return -1

    def setNumber(self, number):
        if self.selectedCell is None:
            return

        rowIdx, columnIdx = self.selectedCell
        cell = self.board[rowIdx][columnIdx]

        if cell.editable:
            cell.number = number

    def delete(self):
        if self.selectedCell is None:
            return
        self.setNumber(0)

    def moveSelectedCell(self, rowMove, columnMove):
        if self.selectedCell is None:
            return

        oldRowIdx, oldColumnIdx = self.selectedCell

        newRowIdx = oldRowIdx + rowMove
        newColumnIdx = oldColumnIdx + columnMove

        if -1 < newRowIdx and newRowIdx < 9 and -1 < newColumnIdx and newColumnIdx < 9:
            self.selectedCell = (newRowIdx, newColumnIdx)
            self.board[oldRowIdx][oldColumnIdx].selected = False
            self.board[newRowIdx][newColumnIdx].selected = True

    def solveSudoku(self):
        self.board = self.getBoard()
        self.solveSudokuHelper(0, 0)

    def solveSudokuHelper(self, rowIdx, columnIdx):
        lastColumnIdx = len(self.board[rowIdx]) - 1
        lastRowIdx = len(self.board) - 1

        currentCell = self.board[rowIdx][columnIdx]
        cellCenterY = self.cellCenters[rowIdx]
        cellCenterX = self.cellCenters[columnIdx]

        if currentCell.number != 0:

            if columnIdx == lastColumnIdx and rowIdx == lastRowIdx:
                return True
            elif columnIdx == lastColumnIdx:
                return self.solveSudokuHelper(rowIdx + 1, 0)
            else:
                return self.solveSudokuHelper(rowIdx, columnIdx + 1)

        currentCell.underConsideration = True

        for candidateNumber in range(1, 10):
            currentCell.number = candidateNumber
            self.drawBoard()
            time.sleep(0.02)
            if self.isValid(candidateNumber, rowIdx, columnIdx):

                if columnIdx == lastColumnIdx and rowIdx == lastRowIdx:
                    currentCell.underConsideration = False
                    return True

                if columnIdx == lastColumnIdx:
                    currentCell.underConsideration = False
                    result = self.solveSudokuHelper(rowIdx + 1, 0)
                else:
                    currentCell.underConsideration = False
                    result = self.solveSudokuHelper(rowIdx, columnIdx + 1)

                if result:
                    currentCell.underConsideration = False
                    return True

            currentCell.underConsideration = True
            self.drawBoard()
            time.sleep(0.02)
            currentCell.number = 0

        currentCell.underConsideration = False
        return False

    def isValid(self, number, rowIdx, columnIdx):
        for idx in range(9):
            if self.board[idx][columnIdx].number == number and idx != rowIdx:
                return False
            if self.board[rowIdx][idx].number == number and idx != columnIdx:
                return False

        squareStartRowIdx = (rowIdx // 3) * 3
        squareStartColumnIdx = (columnIdx // 3) * 3

        for boardRowIdx in range(squareStartRowIdx, squareStartRowIdx + 3):
            for boardColumnIdx in range(squareStartColumnIdx, squareStartColumnIdx + 3):
                if boardRowIdx == rowIdx and boardColumnIdx == columnIdx:
                    continue

                if self.board[boardRowIdx][boardColumnIdx].number == number:
                    return False

        return True


def main():
    # Create a new Pydoku GUI
    pydoku = PydokuGUI()
    pydoku.drawBoard()

    # Set containing the allowed inputs
    ALLOWED_INPUTS = {pygame.K_1, pygame.K_2, pygame.K_3, pygame.K_4,
                      pygame.K_5, pygame.K_6, pygame.K_7, pygame.K_8, pygame.K_9}

    # Run until the user asks to quit
    running = True
    while running:

        # Handle events
        for event in pygame.event.get():
            if event.type == pygame.KEYDOWN:
                if event.key in ALLOWED_INPUTS:
                    number = int(chr(event.key))
                    pydoku.setNumber(number)
                elif event.key == pygame.K_DELETE or event.key == pygame.K_BACKSPACE:
                    pydoku.delete()
                elif event.key == pygame.K_UP:
                    pydoku.moveSelectedCell(-1, 0)
                elif event.key == pygame.K_DOWN:
                    pydoku.moveSelectedCell(1, 0)
                elif event.key == pygame.K_RIGHT:
                    pydoku.moveSelectedCell(0, 1)
                elif event.key == pygame.K_LEFT:
                    pydoku.moveSelectedCell(0, -1)
                elif event.key == pygame.K_SPACE:
                    pydoku.solveSudoku()
                # Draw the actual board
                pydoku.drawBoard()
            if event.type == pygame.MOUSEBUTTONUP:
                pos = pygame.mouse.get_pos()
                pydoku.setSelectedCell(pos)
                # Draw the actual board
                pydoku.drawBoard()

            if event.type == pygame.QUIT:
                running = False

    # Done! Time to quit.
    pygame.quit()


main()
