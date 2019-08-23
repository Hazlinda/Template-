# Template

This template we can use to any new sytem veriilog verification environment.

# DUT
/////////
/////////
// DUT //
/////////
/////////

module adder (
       input clk,
       input reset'
       input //"to be fill follow specification"
       input //"to be fill follow specification"
       input //"to be fill follow specification"
       output //"to be fill follow specification"

reg //"to be fill follow specification"

//////////
//reset///
//////////
always @ (posedge reset)
	//tmp_c<=0;
	//"to be fill follow specification"

/////////////////////	
//addition operator//
/////////////////////
always @ (posedge clk)
	//if (valid) tmp_c <=a+b;
	//"to be fill follow specification"

	//assign c = tmp_c;
	//"to be fill follow specification"

endmodule





# Interface
////////////////
////////////////
//interface.sv//
////////////////
////////////////

interface intf(input logic clk,reset);

////////////////////////
//declaring the signal//
////////////////////////
logic //"to be fill follow specification"
logic //"to be fill follow specification"
logic //"to be fill follow specification"
logic //"to be fill follow specification"

endinterface


//////////////////
//////////////////
//transaction.sv//
//////////////////
//////////////////

class transaction;

///////////////////////////////////
//declaring the transaction items//
///////////////////////////////////
  rand bit //"to be fill follow specification"
  rand bit //"to be fill follow specification"
       bit //"to be fill follow specification"

  function void display(string name);
	  $display("%s", name);
	  $display("a = %0d, b = %0d",a,b);
	  $display("c = %0d", c);
  endfunction
endclass



# Generator
//////////////////
//////////////////
//generator.sv////
//////////////////
//////////////////

class generator;

///////////////////////////////////
//declaring the transaction class//
///////////////////////////////////
rand transaction trans;

////////////////////////////////////////////////////
//repeat count to specify number items to generate//
////////////////////////////////////////////////////
int repeat_count

/////////////////////////////////////////////////////
//mailbox to generate and send the packet to driver//
/////////////////////////////////////////////////////
mailbox gen2driv;

///////////////////////////////////////////
//event to indicate the end of trasaction//
///////////////////////////////////////////
event ended;

//////////////////
//constructor ////
//////////////////

//getting the mailbox handle from env, in order
//to share the transaction packet between the generator 
//and driver, the same mailbox is shared between both.

function new (mailbox gen2driv); 
	this.gen2driv = gen2driv;
endfunction

////////
//Task//
////////

task main();
	repeat(repeat_count) begin
		trans = new();
		if( !trans.randomize()) $fatal("Gen::trans randomization failed");
		trans.display("[Generator]");
		gen2driv.put(trans);
	end
	->ended; //trigering indicates the end of generation
endtask
endclass






# Driver
//////////////
//////////////
//Driver.sv///
//////////////
//////////////

class driver;

////////////////////////////////////////////
//used to count the number of transactions//
////////////////////////////////////////////

int no_transaction;

/////////////////////////////////////
//creating virtual interface handle//
/////////////////////////////////////

virtual intf vif;

///////////////////////////
//creating mailbox handle//
///////////////////////////
mailbox gen2driv;

///////////////
//constructor//
///////////////

function new(virtual intf vif, mailbox gen2driv);
	this.vif=vif; //getting the interface
	this.gen2driv = gen2driv; //getting the mailbox handles from environment
endfunction

/////////////////////////////////////////////////////////////////////
//Reset task, Reset the Interface signals to default/initial values//
/////////////////////////////////////////////////////////////////////
task reset;
	wait (vif.reset);
	$display("[DRIVER] ---Reset Started---");
	//"to be fill follow specification"vif.a <= 0;
	//"to be fill follow specification"vif.b <= 0;
	//"to be fill follow specification"vif.valid <= 0;
	wait(!vif.reset);
	$display("[DRIVER] ---Reset Ended---");
endtask

//////////////////////////////////////////////////////
//drivers the transaction items to interface signals//
//////////////////////////////////////////////////////
task main;
	forever begin
		transaction trans;
		gen2driv.get(trans);
		@(posedge vif.clk);
		//"to be fill follow specification"vif.valid <= 1;
		//"to be fill follow specification"vif.a <= trans.a;
		//"to be fill follow specification"vif.b <= trans.b;
		@(posedge vif.clk);
		//"to be fill follow specification"vif.valid <= 0;
		//"to be fill follow specification"trans.c = vif.c;
		@(posedge vif.clk);
		trans.display("[Driver]");
		no_transaction++;
	end
endtask
endclass






# monitor
////////////////
////////////////
//monitor.sv////
////////////////
////////////////

class monitor;

/////////////////////////////////////
//creating virtual interface handle//
/////////////////////////////////////
virtual intf vif;

///////////////////////////
//creating mailbox handle//
///////////////////////////
mailbox mon2scb;

///////////////
//constructor//
///////////////
function new(virtual intf vif, mailbox mon2scb);
	this.vif = vif;  //getting the iterface
	this.mon2scb = mon2scb;  //getting the mailbox handles from environment
endfunction

//////////////////////////////////////////////////////////////////
//Samples the interface signal and send the packet to scoreboard//
//////////////////////////////////////////////////////////////////
task main;
	forever begin
		transaction trans;
		trans = new();
		@(posedge vif.clk);
		wait(vif.valid);
		//"to be fill follow specification"trans.a = vif.a;
		//"to be fill follow specification"trans.b = vif.b;
		@(posedge vif.clk);
		//"to be fill follow specification"trans.c = vif.c;
		@(posedge vif.clk);
		mon2scb.put(trans);
		trans.display("[Monitor]");
	end
endtask
endclass




# Scoreboard
///////////////////
///////////////////
//scoreboard.sv////
///////////////////
///////////////////

class scoreboard;

///////////////////////////
//creating mailbox handle//
///////////////////////////
mailbox mon2scb;

////////////////////////////////////////////
//used to count the number of transactions//
////////////////////////////////////////////
int no_transaction;

///////////////
//constructor//
///////////////
function new(mailbox mon2scb);
	this.mon2scb = mon2scb;//getting the mailbox handles from  environment 
endfunction

///////////////////////////////////////////////////////
//Compares the Actual result with the expected result//
///////////////////////////////////////////////////////
task main;
	transaction trans;
	forever begin
		mon2scb.get(trans);
		if((trans.a+trans.b) == trans.c);
		$display("Result is as Expected");
	else
		$error("Wrong Result.\n\tExpeced: %0d Actual: %0d", (trans.a+trans.b),trans.c);
	no_transaction++;
	trans.display("[Scoreboard]");
end
endtask
endclass




# Environment
//////////////////
//////////////////
//Environment.sv//
//////////////////
//////////////////

`include "transaction.sv"
`include "generator.sv"
`include "driver.sv"
`include "monitor.sv"
`include "scoreboard.sv"
class environment;

/////////////////////////////////////
///generator and driver instance/////
/////////////////////////////////////
generator gen;
driver driv;
monitor mon;
scoreboard scb;

///////////////////////////////
///mailbox handles/////////////
///////////////////////////////
mailbox gen2driv;
mailbox mon2scb;

////////////////////////////////
////virtual interface///////////
////////////////////////////////
virtual intf vif;

/////////////////////////////////
////constructor//////////////////
/////////////////////////////////
	function new(virtual intf vif);
		this.vif = vif;  //get the interface from test
	
		//////////////////////////////////////
		////creating the mailbox /////////////
		//////////////////////////////////////
		gen2driv = new();
		mon2scb = new();

		//////////////////////////////////////
		////creating generator and driver////
		//////////////////////////////////////
		gen  = new(gen2driv);
		driv = new(vif, gen2driv);
		mon  = new(vif, mon2scb);
		scb  = new(mon2scb);

	endfunction

	task pre_test();
		driv.reset();
	endtask

	task test();
		fork
			gen.main();
			driv.main();
			mon.main();
			scb.main();
		join_any
	endtask
	
////////////////////////////
///////run task/////////////
////////////////////////////
	task run;
		pre_test();
		test();
		post_test();
		$finish;
	endtask
	
endclass




# Testbench
////////////////
////////////////
//testbench.sv//
////////////////
////////////////

////////////////////////
//including interface //
////////////////////////
`include "interface.sv"
`include "random_test.sv"

module tbench_top;

//////////////////////////////////////
//clock and reset signal declaration//
//////////////////////////////////////
bit clk;
bit reset;

////////////////////
//clock generation//
////////////////////
always #5 clk= ~clk;

////////////////////
//reset generation//
////////////////////
initial begin
	reset = 1;
	#5 reset =0;
end

//////////////////////////////////
//creating instance of interface//
//////////////////////////////////
intf i_intf(clk,reset);  

/////////////////////////////////
//creating instance of testcase//
/////////////////////////////////
test t1(i_intf);

/////////////////////////
//creating DUT instance//
/////////////////////////
adder DUT (
	.clk(i_intf.clk),
	.reset(i_intf.reset),
	//"to be fill follow specification".a(i_intf.a),
	//"to be fill follow specification".b(i_intf.b),
	//"to be fill follow specification".valid(i_intf.valid),
	//"to be fill follow specification".c(i_intf.c)
);

endmodule
