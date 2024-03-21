## **🎴21点（Blackjack）扑克游戏**

>
#### **目标**
>
>玩家的手牌点数尽量接近21点，但不能超过21点。
同时要求玩家手牌点数高于庄家，即玩家争取在不超过21点的情况下，>手牌点数尽可能地比庄家高。

>
#### **牌点计算**
>
>1. 2至10的牌按其牌面点数计算。
>3. J、Q、K每张牌的点数为10点。
>3. A牌可记为1点或11点，根据有利原则来决定。

>
#### **游戏流程**

>1. 玩家下注，游戏开始。
>2. 庄家给每位玩家发两张牌，庄家自己也发两张牌，其中一张牌面向上，另一张牌面向下。
>3. 玩家根据手牌点数决定是否要牌（Hit）或停牌（Stand）。
>3. 要牌：再要一张牌，可反复要牌直到决定停牌或爆牌。
>3. 停牌：不再要牌，等待庄家行动。
>3. 如果玩家手牌超过21点称为"爆牌"（Bust），立即输掉本局游戏。
>3. 玩家停牌后，庄家亮出暗牌并根据规则要牌或停牌：
>3. 庄家手牌点数小于17点，必须要牌。
>3. 庄家手牌点数大于等于17点，必须停牌。
>3. 比较玩家和庄家的手牌点数，点数较高的一方获胜。
>3. 玩家获胜，赢得与本金相同的筹码。
>3. 庄家获胜，玩家输掉本金筹码。
>3. 点数相同（和局），玩家收回本金筹码。

>
#### **特殊情况**

>1. Blackjack：玩家前两张牌为一张A牌和一张10点牌，称为"Blackjack"，直接获胜并赢得1.5倍本金。
>3. 分牌（Split）：玩家前两张牌点数相同时，可选择分牌，分成两手牌分别进行游戏，需额外下注一倍本金。
>3. 双倍下注（Double Down）：玩家前两张牌时，可选择双倍下注，并且只能再要一张牌。
>3. 保险（Insurance）：当庄家亮出的牌为A时，玩家可选择保险，额外下注一半本金。如果庄家为Blackjack，保险赢得2倍保险金；否则，保险输掉。

以上就是21点的基本游戏规则，部分规则在不同的赌场或游戏变体中可能略有不同。

>下方是我所写的完整代码，部分规则并没有实现。两个人玩。

```Python

import random
from abc import ABCMeta, abstractmethod
import time
import sys

class Card(object):
    '''
    创建一个牌类，记录牌面以及对应的牌分
    '''

    def __init__(self, face):
        '''
        初始化，给属性牌面赋值（1-14）
        '''
        self.face = face

    def get_value(self):
        '''
        返回牌面对应的牌分
        '''
        values = ['', [1, 11], 2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10]
        return values[self.face]
    
    def __repr__(self):
        '''
        返回一个牌面（牌分）字符串
        '''
        faces = ['', 'A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K']
        values = ['', [1, 11], 2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10]
        return f'{faces[self.face]}({values[self.face]})'

# test_Card
# crd = Card(12)
# print(crd)
# print(crd.get_value())

class Deck(object):
    '''
    创建一个牌组类，记录牌组，用于发牌，洗牌，判断牌数
    '''

    def __init__(self, decks = 1):
        '''
        初始化，参数包括牌组数目（几副牌）
        '''
        self.cards = [Card(face) for deck in range(1, decks+1) for i in range(4) for face in range(1,14)]
        ## 根据牌组数目生成 卡牌叠
        self.point = 0
        ## 指针，用于指向当前牌的位置

    def shuffle(self):
        self.point = 0
        random.shuffle(self.cards)
        ## 随机洗牌

    def licensing(self):
        '''
        发牌，每次发一张,指针向后退一个
        '''
        card = self.cards[self.point]
        self.point += 1
        return card

    def is_enough(self):
        '''
        还够发吗？默认规定还剩5张牌就不再发了
        '''
        return len(self.cards) - self.point <= 5

# test_Deck 
# dk = Deck(2)
# print(dk.cards)
# dk.shuffle()
# print(len(dk.cards))
# print(dk.cards)
# print(dk.licensing())
# print(dk.is_enough())

class Player(metaclass = ABCMeta):

    def __init__(self, name, chip = 100):
        self.name = name
        self.score = 0  ## 牌分总和，每次摸牌后都会变化，用于评判
        self.chip = chip  ## 筹码，默认100
        self.cards = []

    @abstractmethod  
    def Draw(self):
        '''
        摸牌，一次摸一张牌，并获取其牌分，记录在牌分库里
        '''
        pass

    def Bet(self,num):
        '''
        下注
        '''
        self.chip -= num
        return self.chip
    
    def Scoring(self):
        '''
        计分，每摸一次牌后算一次分
        '''
        if self.cards[-1].face != 1:
            self.score += self.cards[-1].get_value()
        else:
            if self.score + 11 > 21:
                self.score += self.cards[-1].get_value()[0]  ## 总分+11后大于21，则加1
            else:
                self.score += self.cards[-1].get_value()[1]  ## 总分+11后小于21，则加11
        return self.score  ## 返回此次摸牌后分数

class Makers(Player):  ## 庄家

    def __init__(self, name = 'Robot', chip = 100):
        self.name = name
        self.score = 0
        self.chip = chip
        self.cards = []

    def Draw(self, card):
        self.cards.append(card) 

class Downs(Player):  ## 下家

    def __init__(self, name, chip = 100):
        self.name = name
        self.score = 0
        self.chip = chip
        self.cards = []

    def Draw(self, card):
        self.cards.append(card) 

# test_Player
# dk = Deck()
# dk.shuffle()
# print(dk.cards)
# jim = Downs('jim')
# jim.Bet(5)
# jim.Draw(dk.licensing())
# print(jim.Scoring())
# jim.Draw(dk.licensing())
# print(jim.Scoring())
# jim.Draw(dk.licensing())
# print(jim.Scoring())
# jim.Draw(dk.licensing())
# print(jim.Scoring())
# print(jim.cards)
# print(jim.chip)
# print(jim.score)

class Game(object):
    '''
    牌局
    '''
    
    def __init__(self):
        
        print('欢迎来到Blackjorker游戏！')
        print()

        ## 创建游戏玩家
        self.init_chip = int(input('请输入初始玩家筹码数目：'))
        self.player1 = Makers(chip = self.init_chip) ## 庄家
        self.player2 = Downs(name = input('请输入用户名创建您的游戏角色：'), chip = self.init_chip)  ## 玩家（下家）
        self.players = [self.player1, self.player2]

        ## 其它牌局参数
        self.bet_chip = 0  ## 玩家下注
        self.count = 1  ##游戏轮数
        self.times = 1  ##初始倍率
        self.div = 1  ##回合

        ## 输出牌局信息
        print()
        print('所有玩家都已进入牌局，请查看你们的信息：')
        print(f'庄家{self.player1.name}，初始筹码{self.player1.chip}')
        print(f'庄家{self.player2.name}，初始筹码{self.player2.chip}')

        ## 创建牌组资源
        print()
        self.decks = Deck(int(input('请输入牌组数目：')))  ## 创建牌组
        self.decks.shuffle()  ##洗牌

        ## 准备开始游戏
        print()
        flag = int(input('牌局准备完毕！请输入1开始游戏/0退出游戏：'))
        if flag == 1:
            self.GameStart()
        else:
            sys.exit()

    def Judge(self):
        '''
        0 继续
        1 玩家1胜利
        2 玩家2胜利
        3 平局
        '''
        if self.div == 1:
            if self.player1.score == 21 and self.player1.score != self.player2.score: ## 庄家是BJ
                self.times = 1.5
                return 1
            elif self.player2.score == 21 and self.player1.score != self.player2.score:  ## 玩家是BJ
                self.times = 1.5
                return 2
            else:
                return 0
        elif self.div == 2:
            if self.player2.score > 21:
                return 1
            else:
                return 0
        else:
            if self.player1.score > 21:
                return 2
            elif self.player1.score > self.player2.score:
                return 1
            elif self.player2.score > self.player1.score:
                return 2
            elif self.player1.score == self.player2.score:
                return 3

    def  Count(self, ord):
        if ord == 1:
            self.player1.chip = self.player1.chip + self.times * self.bet_chip
            self.player2.chip -= (self.times - 1) * self.bet_chip
        elif ord == 2:
            self.player2.chip = self.player2.chip + self.bet_chip + self.times * self.bet_chip
            self.player1.chip -= self.times * self.bet_chip
        elif ord == 3:
            self.player2.chip = self.player2.chip + self.bet_chip
    
    def GameStart(self):

        ## 牌局正式开始
        while self.player1.chip > 0 and self.player2.chip > 0 and not(self.decks.is_enough()):

            print()
            self.times = 1 ## 倍率置1
            self.player1.cards = []
            self.player2.cards = []
            self.player1.score = 0
            self.player2.score = 0
            self.div = 1  ## 记录当前轮次中的回合
            
            print(f'第{self.count}轮游戏开始，请玩家{self.player2.name}下注！')
            self.count += 1 ## 轮数+1
            
            self.bet_chip = int(input('请输入下注筹码数额：'))  ## 下注筹码放在牌局上
            self.player2.Bet(self.bet_chip)
            print(self.player2.chip) ## test
            
            print()
            print('下注环节结束，接下来每人将会发到两张底牌！')

            ## 发底牌
            for i in range(2):
                for player in self.players:
                    player.Draw(self.decks.licensing())  ## 摸牌
                    player.Scoring()  ## 计算牌分

            ## 底牌展示
            time.sleep(1)
            print('底牌已发完，请查看自己当前手牌：')
            print(f'庄家{self.player1.name}明牌：[{self.player1.cards[random.randint(0, 1)]}]')
            print(f'玩家{self.player2.name}手牌：{self.player2.cards}')

            ##判断首轮输赢
            print()
            print('正在判断本轮胜负情况...')
            time.sleep(1)
            win_flag = self.Judge()
            
            if win_flag == 1:
                print(f'庄家{self.player1.name}本轮胜出！')
                self.Count(self.Judge())
            elif win_flag == 2:
                print(f'玩家{self.player2.name}本轮胜出！')
                self.Count(self.Judge())
            else:
                while True:
                    
                    flag = int(input('您的回合，是否要牌？1（要牌）/0（停牌）：'))
                    if flag == 1:
                        self.player2.Draw(self.decks.licensing())
                        self.player2.Scoring()
                        print(f'玩家{self.player2.name}手牌：{self.player2.cards}')
                        
                        ##判断输赢
                        print()
                        print('正在判断本轮胜负情况...')
                        time.sleep(1)
                        self.div = 2
                        win_flag = self.Judge()

                        if win_flag == 1:
                            print(f'庄家{self.player1.name}本轮胜出！')
                            self.Count(self.Judge())
                            break
                    
                    else:
                        print()
                        print('庄家的回合')
                        while self.player1.score < 17:
                            print('庄家要牌')
                            self.player1.Draw(self.decks.licensing())
                            self.player1.Scoring()

                        ##判断输赢
                        print()
                        print('正在判断本轮胜负情况...')
                        time.sleep(1)
                        self.div = 3
                        win_flag = self.Judge()
                        
                        if win_flag == 1:
                            print(f'庄家{self.player1.name}本轮胜出！')
                            self.Count(self.Judge())
                            break
                        elif win_flag == 2:
                            print(f'玩家{self.player2.name}本轮胜出！')
                            self.Count(self.Judge())
                            break
                        elif win_flag == 3:
                            print(f'庄家{self.player1.name}和玩家{self.player1.name}平局！')
                            self.Count(self.Judge())
                            break
                        
            
            print()
            print('本轮结束，即将进入下一轮，双方剩余筹码数目：')
            print(f'庄家{self.player1.name}，剩余筹码{self.player1.chip}')
            print(f'玩家{self.player2.name}，剩余筹码{self.player2.chip}')
            
                        
        ## 牌局结束，清点筹码
        print()
        print('Blackjorker游戏结束，欢迎各位参与，以下是本场牌局筹码情况：')
        print(f'庄家{self.player1.name}，结算筹码{self.player1.chip}')
        print(f'玩家{self.player2.name}，结算筹码{self.player2.chip}')

        print('- THE END -')
    

NewGame = Game()

```
