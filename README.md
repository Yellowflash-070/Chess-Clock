# Chess-Clock
A two player digital chess clock system used to keep track of time in hrs:mins:sec taken by each player while making their respective move during the passage of the game.

<br>

## Components of the Chess Clock System
1. Two timers, each for players A and B.
2. The main Chess Clock controller FSM

<br>

## Functionality of the Chess Clock System

Whenever a new game begins, the control is initially in the (Stop_Wait) state and both the timers A and B hold their respective values (which is 0 secs when the game starts).

Whenever a player (Let's say Player A) is to make the first move, then Player B needs to start the game by pressing his/her push button (Pb). This will force start Player A's timer by asserting (Ta) signal in order to record time taken by Player A to make the first move. The control at this moment is in (RunA) state.

Whenever Player A makes his/her move, he/she needs to press his/her push button (Pa). This will force stop Player A's timer by deasserting (Ta) signal which in turn force starts Player B's timer by asserting (Tb) signal in order to record time taken by Player B to make their move. The control at this moment is in (RunB) state.

In case if both the Players simultaneously press their push buttons, the system is designed in such a way that it holds the timer of both Players to its previous value until a single Player makes the move. The control at this moment again goes back to (Stop_Wait) state.

<br>

## Verilog code for main module

``` verilog
module chess_clock_fsm(clk,pa,pb,ta,clr,tb,rst);
input clk,pa,pb,rst;
output reg ta,tb,clr;


reg [1:0] state, next_state;

parameter Stop_Wait = 2'b00;
parameter RunA = 2'b01;
parameter RunB = 2'b10;



//initializing varibles
    initial
        begin
            ta = 0;
            tb = 0;
            clr = 0;
            state = 0;
            next_state = 0;
        end


//State register logic
    always @(posedge clk, posedge rst)
        if(rst)
            begin
                state <= Stop_Wait;
                clr <= 1;
            end
        else
            begin
                clr <= 0;
                state <= next_state;
            end
            

//next-state logic
    always @(state,pa,pb)
        begin
            case(state)
                Stop_Wait: if(pa && pb)
                        begin
                            ta = 0;
                            tb = 0;
                            next_state = Stop_Wait;
                        end
                      else if(pa && ~pb)
                        begin
                            tb = 1;
                            next_state = RunB;
                        end    
                      else if(~pa && pb)
                        begin
                            ta = 1;
                            next_state = RunA;
                        end
                      else
                            next_state = Stop_Wait;
           
                RunA : if(pa && pb)
                       begin
                             ta = 0;
                             tb = 0;
                             next_state = Stop_Wait;
                        end 
                       else if(pa)
                        begin
                            ta = 0;
                            tb = 1;
                            next_state = RunB;
                        end
                       else if(pb)
                            next_state = RunA;
                       else
                            next_state = Stop_Wait;
                        
                         
                RunB : if(pa && pb)
                       begin
                             ta = 0;
                             tb = 0;
                             next_state = Stop_Wait;
                        end 
                       else if(pb)
                        begin
                            tb = 0;
                            ta = 1;
                            next_state = RunA;
                        end
                       else if(pa)
                            next_state = RunB;
                       else
                            next_state = Stop_Wait;
                        
                               
                default: next_state = Stop_Wait;
                                  
            endcase
        end
        
//timer instantiation
timer TA(.clk(clk), .en(ta), .rst(clr));
timer TB(.clk(clk), .en(tb), .rst(clr));


endmodule

```

<br>

## Verilog code for Timer module

``` verilog
module timer(en,rst,clk);
input en,rst,clk;

reg [5:0]hrs,min,sec;

initial
    begin
        hrs = 0;
        min = 0;
        sec = 0;
    end

always @(posedge clk, posedge rst)
    if(rst)
        begin
            hrs <= 0;
            min <= 0;
            sec <= 0;
        end
    else
        begin
            if(en)
                begin
                    if(sec >= 60)
                        begin
                            sec <= 0;
                            min <= min + 1;
                        end
                    else
                        sec <= sec + 1;
                        
                    if(min >= 60)
                        begin
                            hrs <= hrs + 1;
                            min <= 0;
                        end
                    else
                        if(sec >= 60)
                            min <= min + 1;
                end
            else
                begin
                    $display ("hours = %d", hrs);
                    $display ("minutes = %d", min);
                    $display ("seconds = %d", sec);
                end
        end
endmodule

```

<br>

## Testbench to check the functionality of our system

``` verilog
module chess_clock_tb;

reg clk,pa,pb,rst;
wire ta,tb,clr;


chess_clock_fsm cc(
    
    .clk(clk),.pa(pa),.pb(pb),
    .ta(ta),.clr(clr),.tb(tb),
    .rst(rst)        

);


initial 
    begin
        clk = 1'b0;
        pa = 1'b0;
        pb = 1'b0;
        rst = 1'b0;
    end
    
always #5 clk = ~clk;
    
    
initial
    begin
    
        rst = 1'b1;
      
      #2 rst = 1'b0;
      
      #2 pa = 1'b0;
         pb = 1'b0;
          
      #50 pa = 1'b1;
      
      #80 pb = 1'b1;
           pa = 1'b0;
          
      #80 pa = 1'b1;
          
      #50 pa = 1'b0;
          pb = 1'b1;
           
      #50 pa = 1'b1;
          pb = 1'b0;
          
      #30 rst = 1'b1;
           
      #30 $finish;    
                 
    end
    
     
endmodule

```

<br>

## Simulation Waveform

![Screenshot 2023-12-23 164353](https://github.com/Yellowflash-070/Chess-Clock/assets/153358226/67da72d8-ff51-4662-a688-9c8035653729)

<br>

* In the following simulation snapshot, initially both (Pa) and (Pb) signal are held low, so both the timers of player A and B hold their value which in this case is 0 sec as the game has just begun.

* When (Pa) goes high and (Pb) is held low, signal (Tb) is asserted which in turn force starts Player B's timer.

* When (Pb) goes high and (Pa) is goes low, signal (Ta) is asserted which in turn force starts Player A's timer and Player B's timer stops and holds its current value.

* In case if both (Pa) and (Pb) goes high , then both the timers of Player A and Player B stops and holds their current value.

* In case if a reset signal (rst) is asserted the value on both the timer gets reset to zero.


<br>

**NOTE: All coding and simulations are performed in Vivado Design suite 2020.2 edition.**

  


