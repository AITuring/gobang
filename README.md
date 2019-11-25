## gobang
一个简单的在线五子棋对战AI
欢迎访问http://go.loveai.xyz

### 代码结构
本实验是由html、css以及javascript构成的，其中html和css主要规定了棋盘的样式。主要代码在JavaScript中。js中，代码主要有两部分构成，一部分是绘制棋盘，棋子以及实现鼠标点击落子。另一部分是利用搜索算法实现机器落子与人对战。
#### 五子棋的实现
这一部分主要有三个函数：

1. drawChessBoard利用循环绘制直线从而绘制整个棋盘。

2. oneStep绘制渐变的棋子。其中context.arc()的四个参数分别是圆心横坐标、圆心纵坐标、半径、开始弧度、结束弧度。context.createRadialGradient()定义了一个有渐变的颜色变量，前三个参数是一个圆的圆心坐标与半径，后三个参数是另一个圆的圆心坐标与半径。
3. gobang.onclick在画布上绑定单击事件来实现用鼠标点击棋盘就落下一颗棋子。通过e.offsetX和e.offsetY两个属性得到坐标，后转化成i和j，再调用oneStep()方法，定义一个变量chess来决定是黑子还是白子，每点击一次就改变一次chess的值。但是已经下了黑子的点，重新点击还会被白子覆盖掉。这里用了二维数组GobangBoard放所有的落子点，如果有落子，就给其记录下来。落子的时候再判断是否已经落子，如果已经落子了就不允许重新落子。二维数组的初始值都是0，然后在单击事件的方法里添加一个判断，落子位置等于0才可以落子，落完子后给相应的点附非0值。

#### 五子棋AI的实现
首先定义一个三维数组wins，用于存放所有赢的情况。定义count用于记录赢的方法种类数量，然后再遍历所有赢的情况，包括横向、纵向、斜向、反斜向。

然后再定义了一个一维的赢法的统计数组用来计算某种赢法是否达成。上面列举了所有的赢法，那在某赢法中成功落下一个棋子，那我们就为该种赢法的值加1，直到加5就实现了此种赢法，实现结束游戏。那如果该赢法中有另一方的棋子，此种赢法报废。

定义并实例化两个数组myWin和computerWin，一个用来存放玩家赢法的数组，另一个用来存放电脑赢法的数组。

定义了一个over变量，标志游戏是否结束。在单击事件，成功落子后进行一个循环。
这个循环遍历所有的赢法，如果某种赢法里有棋子，就myWin[k]++，如果myWin[k] == 5成立了，改变over的标志，就代表赢了。

在完成落子统计后判断是否结束游戏，如果没结束，就轮到计算机下棋，计算棋下棋调用computerAI()方法。在点击鼠标的时候需要判断是否轮到人落棋。更改落棋的判断，如果鼠标点击，只下黑棋，并且只改变黑棋的频数统计。

之后需要计算机判断有效落子点。首先定义并实例化myScore与computerScore两个数组，用于存放各个棋点的落子价值，再分别遍历人和计算机的所有赢法，如果有1、2、3、4子，分别为棋点附权值。
```javascript

var computerAI = function(){
    var myScore = [] ;//定义两个数组，用于存放每个点的分值
    var computerScore = [] ;
    for(var i=0; i<15; i++){
        myScore[i] = [] ;
        computerScore[i] = [] ;
        for(var j=0; j<15; j++){
            myScore[i][j] = 0 ;
            computerScore[i][j] = 0;
        }
    }
    //为所有赢法打分
    for(var i=0; i<15; i++){
        for(var j=0; j<15; j++){
            if(chessBoard[i][j] == 0){//该点可下棋
                for(var k=0; k<count; k++){//遍历所有赢法
                    if(wins[i][j][k]){//这种赢法已经有子
                        if(myWin[k] == 1){//黑方，有1个子
                            myScore[i][j] += 200 ;
                        }else if(myWin[k] == 2){
                            myScore[i][j] += 400 ;
                        }else if(myWin[k] == 3){
                            myScore[i][j] += 2000 ;
                        }else if(myWin[k] == 4){
                            myScore[i][j] += 10000 ;
                        }
                        if(computerWin[k] == 1){//白方，有1个子
                            computerScore[i][j] += 220 ;
                        }else if(computerWin[k] == 2){
                            computerScore[i][j] += 420 ;
                        }else if(computerWin[k] == 3){
                            computerScore[i][j] += 2200 ;
                        }else if(computerWin[k] == 4){
                            computerScore[i][j] += 20000 ;
                        }
                    }
                }
            }
        }
    }
}
```

接下来找到计算机落子的最佳坐标，然后在这个坐标上落子，就实现了计算机落子的功能。为了找到棋盘坐标最高权值和其坐标点，这里定义三个变量max、u和v 。通过上遍历所有赢法的值后，这里只要判断比较在哪一方的赢法中落子价值大，主要是自己向着赢的方向前进和不让对方赢的方向进行阻挡。
```javascript
if(myScore[i][j] > max ){//人在某步的权值更高的时候
    max = myScore[i][j] ;
    u = i ;
    v = j ;
}else if(myScore[i][j] = max){//如果权值是最大了
    if(computerScore[i][j] > computerScore[u][v]){//而i,j点的权值比在u,v点的更大时
        u = i ;
        v = j ;
    }
}
if(computerScore[i][j] > max ){//人在某步的权值更高的时候
    max = computerScore[i][j] ;
    u = i ;
    v = j ;
}else if(computerScore[i][j] = max){//如果权值是最大了
    if(myScore[i][j] > myScore[u][v]){//而计算机在此处落子更有用
        u = i ;
        v = j ;
    }
}
```
得到最大权值的坐标后，我们就要实现在该位置落子：
落完子后要对落子的位置进行统计，如果达到赢的情况就终止游戏，否则把落子权交给鼠标。
```javascript
oneStep(u, v, false) ;//上面得到最佳落子点，现在为其落白子
GobangBoard[u][v] == 2 ; 
 //计算机落完子后需要进行统计
for(var k=0; k<count; k++){
    if(wins[u][v][k]){//某种赢的某子true
        computerWin[k]++;//离胜利又进一步
        myWin[k] = 6 ;//该种赢法计算机没有机会了
        if(computerWin[k] == 5){//如果达到5就赢了
            window.alert("计算机赢了！！");
            over = true ;
        }
    }
}
if(!over){
    chess = !chess ;  //如果游戏没结束，
}
```
### 5.实验结果展示
欢迎访问 http://go.loveai.xyz
