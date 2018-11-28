module Phase_2(clk, rst, key, start_game, DAC_clk, VGA_R, VGA_G, VGA_B, VGA_Hsync, 
					VGA_Vsync, blank_n, KB_clk, data);
					
input clk, rst;
input KB_clk, data;
input [1:0]key;
input start_game;

wire [2:0]direction;
wire reset;

output reg [7:0]VGA_R;
output reg [7:0]VGA_G;
output reg [7:0]VGA_B;

output VGA_Hsync;
output VGA_Vsync;
output DAC_clk;
output blank_n;

wire [10:0]xCounter;
wire [10:0]yCounter;

wire R;
wire G;
wire B;

wire update;
wire VGA_clk;
wire displayArea;

wire paddle;
wire ball;
reg border;
reg game_over;
reg win_game;
reg [10:0]x,y; //the top left point of the paddle
reg [10:0]x_ball,y_ball; //the top right of the ball

//import modules
kbInput keyboard(KB_clk, key, direction, reset); //the "keyboard", aka the buttons
updateCLK clk_updateCLK(clk, update);
clk_reduce reduce(clk, VGA_clk);
VGA_generator generator(VGA_clk, VGA_Hsync, VGA_Vsync, DisplayArea, xCounter, yCounter, blank_n);

assign DAC_clk = VGA_clk; //DON'T DELETE. this allows the clock on the board to sync with the vga (allowing things to shop up on the monitor)

assign paddle = (xCounter >= x && xCounter <= x+60 && yCounter >= y && yCounter <= y + 15); // sets the size of the paddle
assign ball   = (xCounter >= x_ball && xCounter <= x_ball+10 && yCounter >= y_ball && yCounter <= y_ball + 10); // sets the size of the ball

always @(posedge update)
begin
	if(rst == 0)
	begin	
		x <= 11'd290; 
		y <= 11'd465;
		x_ball <= 11'd315;
		y_ball <= 11'd454;
	end
	else
	begin
		case(start_game)
			1'd0: y_ball <= y_ball;
			1'd1: y_ball <= y_ball - 11'd20;
		endcase
		case(direction)
			3'd1: x <= x + 11'd10; //left at a speed of "10"
			3'd2: x <= x - 11'd10; //right at a speed of "10"
			default: 
			begin
				x <= x; //stationary
			end
		endcase		
	end
end

always @(posedge VGA_clk) //border and color
begin
	border <= (((xCounter >= 0) && (xCounter < 11) || (xCounter >= 630) && (xCounter < 641)) 
				|| ((yCounter >= 0) && (yCounter < 11) || (yCounter >= 470) && (yCounter < 481)));
	VGA_R = {8{R}};
	VGA_G = {8{G}};
	VGA_B = {8{B}};
end

//assigning colors to objects
assign R = 1'b1 && ~ball;
assign B = 1'b1;
assign G = paddle || ball;
	
endmodule

/////////////////////////////////////////////////////////////////// VGA_generator to display using VGA
module VGA_generator(VGA_clk, VGA_Hsync, VGA_Vsync, DisplayArea, xCounter, yCounter, blank_n);
input VGA_clk;
output VGA_Hsync, VGA_Vsync, blank_n;
output reg DisplayArea;
output reg [9:0] xCounter;
output reg [9:0] yCounter;

reg HSync;
reg VSync;

integer HFront = 640;//640
integer hSync = 655;//655
integer HBack = 747;//747
integer maxH = 793;//793

integer VFront = 480;//480
integer vSync = 490;//490
integer VBack = 492;//492
integer maxV = 525;//525

always @(posedge VGA_clk)
begin		
	if(xCounter == maxH)
	begin
		xCounter <= 0;
		if(yCounter === maxV)
			yCounter <= 0;
		else
			yCounter <= yCounter +1;
	end
	else
	begin
		xCounter <= xCounter + 1;
	end
	DisplayArea <= ((xCounter < HFront) && (yCounter < VFront));
	HSync <= ((xCounter >= hSync) && (xCounter < HBack));
	VSync <= ((yCounter >= vSync) && (yCounter < VBack));
end

assign VGA_Vsync = ~VSync;
assign VGA_Hsync = ~HSync;
assign blank_n = DisplayArea;

endmodule

/////////////////////////////////////////////////////////////////// update clk to lower snake speed
module updateCLK(clk, update);
input clk;
output reg update;
reg[21:0]count;

always @(posedge clk)
begin
	count <= count + 1;
	if(count == 2500000)
	begin
		update <= ~update;
		count <= 0;
	end
end
endmodule

/////////////////////////////////////////////////////////////////// reduce clk from 50MHz to 25MHz
module clk_reduce(clk, VGA_clk);

	input clk;
	output reg VGA_clk;
	reg a;

	always @(posedge clk)
	begin
		a <= ~a; 
		VGA_clk <= a;
	end
endmodule

module kbInput(KB_clk, key, direction, reset);
input KB_clk;
input [1:0]key;
output reg [2:0]direction;
output reg reset = 0; 

always @(KB_clk)
begin
	if(key[1] == 1'b1 & key[0] == 1'b0)
		direction = 3'd1;//left
	else if(key[0] == 1'b1 & key[1] == 1'b0)
		direction = 3'd2;//right
	else
		direction = 3'd0;//stationary
end
endmodule