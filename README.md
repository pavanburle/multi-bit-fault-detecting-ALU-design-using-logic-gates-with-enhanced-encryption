# multi-bit-fault-detecting-ALU-design-using-logic-gates-with-enhanced-encryption

module Encrypted_ALU(A,B,Cin,k1,k2,sel,ALU_out);

input [3:0] A,B;
input Cin,k1,k2;
input [2:0] sel ;
output reg [7:0]ALU_out;

wire [4:0]Yadd;//4SUM+1CARRY
wire [7:0]Yproduct,Yand,Yor,Ynot;
wire [3:0]Ydiff ;
wire Ybrw,alb,aeb,agb ;

RIPPLE_CARRY_ADDER4BIT ins1(.A(A),.B(B),.Cin(Cin),.k1(k1),.k2(k2),.SUM(Yadd[3:0]),.CARRY(Yadd[4]));
Dadda_Multiplier_4bit ins2( .md(A) , .mr(B) ,.k1(k1),.k2(k2), .product(Yproduct) );
SUBTRACTOR ins3(.A(A),.B(B),.DIFF(Ydiff),.k1(k1),.k2(k2),.barrow(Ybrw));
comparator ins4(.A(A),.B(B),.altb(alb),.k1(k1),.k2(k2),.aeqb(aeb),.agtb(agb)) ;
///logical and

assign Yand = (k1==1&&k2==0) ? A & B : ~(A & B) ;

assign Yor = (k1==1&&k2==0) ? A | B :  ~(A | B);

assign Ynot = (k1==1&&k2==0) ? ~ A : A;


always@ *
begin

	case(sel)
	3'b000 : ALU_out = Yadd ;
	3'b001 : ALU_out = Yproduct ;
	3'b010 : ALU_out = Ydiff ;
	3'b011 : ALU_out = {alb,aeb,agb} ;
	3'b100 : ALU_out = Yand ;
	3'b101 : ALU_out = Yor ;
	3'b110 : ALU_out = Ynot ;
	default  ALU_out = 8'd0 ;
	endcase
	
end
endmodule


//////4BIT RIPPLE CARRY ADDER//////////
module RIPPLE_CARRY_ADDER4BIT(A,B,Cin,k1,k2,SUM,CARRY);
input [3:0]A,B;
input Cin,k1,k2 ;
output [3:0]SUM;
output CARRY ;
//full_adder(a,b,c,s,c0);
full_adder full_adder1(.a(A[0]),.b(B[0]),.c(Cin),.k1(k1),.k2(k2),.sum(SUM[0]),.carry(C1));
full_adder full_adder2(.a(A[1]),.b(B[1]),.c(C1),.k1(k1),.k2(k2),.sum(SUM[1]),.carry(C2));
full_adder full_adder3(.a(A[2]),.b(B[2]),.c(C2),.k1(k1),.k2(k2),.sum(SUM[2]),.carry(C3));
full_adder full_adder4(.a(A[3]),.b(B[3]),.c(C3),.k1(k1),.k2(k2),.sum(SUM[3]),.carry(CARRY));


endmodule


///multiplier

module Dadda_Multiplier_4bit( md , mr ,k1,k2, product  );

input [3:0] md , mr ;

input k1,k2 ;

output [7:0] product ;

assign md0mr0 = md[0] & mr[0] ;
assign md1mr0 = md[1] & mr[0] ;
assign md2mr0 = md[2] & mr[0] ;
assign md3mr0 = md[3] & mr[0] ;

assign md0mr1 = md[0] & mr[1] ;
assign md1mr1 = md[1] & mr[1] ;
assign md2mr1 = md[2] & mr[1] ;
assign md3mr1 = md[3] & mr[1] ;

assign md0mr2 = md[0] & mr[2] ;
assign md1mr2 = md[1] & mr[2] ;
assign md2mr2 = md[2] & mr[2] ;
assign md3mr2 = md[3] & mr[2] ;

assign md0mr3 = md[0] & mr[3] ;
assign md1mr3 = md[1] & mr[3] ;
assign md2mr3 = md[2] & mr[3] ;
assign md3mr3 = md[3] & mr[3] ;

//level 1

half_adder  ha1 ( .a(md1mr0) , .b(md0mr1) , .sum(hs1) , .carry(hc1) ) ;
full_adder full_adder1 ( .a(md2mr0) , .b(md1mr1) , .c(md0mr2) ,.k1(k1),.k2(k2), .sum(fs1) , .carry(fc1) ) ;
full_adder full_adder2 ( .a(md2mr1) , .b(md1mr2) , .c(md0mr3) ,.k1(k1),.k2(k2), .sum(fs2) , .carry(fc2) ) ;
full_adder full_adder3 ( .a(md3mr1) , .b(md2mr2) , .c(md1mr3) ,.k1(k1),.k2(k2), .sum(fs3) , .carry(fc3) ) ;

//level 2

half_adder  ha2 ( .a(fs1) , .b(hc1) , .sum(hs2) , .carry(hc2) ) ;
full_adder full_adder4 ( .a(md3mr0) , .b(fs2) , .c(fc1) ,.k1(k1),.k2(k2), .sum(fs4) , .carry(fc4) ) ;
half_adder  ha3 ( .a(fs3) , .b(fc2) , .sum(hs3) , .carry(hc3) ) ;
full_adder full_adder5 ( .a(md3mr2) , .b(md2mr3) , .c(fc3) ,.k1(k1),.k2(k2), .sum(fs5) , .carry(fc5) ) ;

//level 3

half_adder  ha4 ( .a(fs4) , .b(hc2) , .sum(product[3]) , .carry(hc4) ) ;
full_adder full_adder6 ( .a(hs3) , .b(fc4) , .c(hc4) , .k1(k1),.k2(k2),.sum(product[4]) , .carry(fc6) ) ;
full_adder full_adder7 ( .a(fs5) , .b(hc3) , .c(fc6) , .k1(k1),.k2(k2),.sum(product[5]) , .carry(fc7) ) ;
full_adder full_adder8 ( .a(md3mr3) , .b(fc5) , .c(fc7) ,.k1(k1),.k2(k2), .sum(product[6]) , .carry(product[7]) ) ;

assign product[2:0] = { hs2 , hs1 , md0mr0 } ;

endmodule

///half adder

module half_adder ( a , b , sum , carry ) ;

input a , b ;

output sum , carry  ;

assign sum = a ^ b ;

assign carry = a & b ;

endmodule 

///full adder

module full_adder ( a , b , c ,k1,k2, sum , carry ) ;

input a , b , c ,k1,k2;

output sum , carry  ;

//assign sum = a ^ b ^ c ;

//assign carry = ( a & b ) | ( b & c ) | ( a & c ) ;

half_adder ha1( a , b , s1 , c1 ) ;
kg1 kg11(s1,k1,w1);
half_adder ha2( w1 , c , sum , c2 ) ;
kg2 kg22(c1,k2,w2);
assign carry = w2 | c2 ;

endmodule 
//////sub tractor
module kg1(s1,k1,w1);

input s1,k1 ;
output w1 ;

assign w1 = k1 ? s1 : !s1 ;

endmodule

module kg2(c1,k2,w2);

input c1,k2 ;
output w2 ;

assign w2 = k2 ? !c1 : c1 ;

endmodule


module SUBTRACTOR(A,B,k1,k2,DIFF,barrow);

input [3:0]A,B ;
input k1,k2 ;
output [3:0]DIFF ;
output barrow ;

wire [3:0]B1 ;
//2's complement for B
assign B1 = ~B + 1 ;
//full_adder(a,b,c,K0,K1,s,c0);
//NAMEED
full_adder full_adder1(.a(A[0]),.b(B1[0]),.c(1'b0),.k1(k1),.k2(k2),.sum(DIFF[0]),.carry(C1));
full_adder full_adder2(.a(A[1]),.b(B1[1]),.c(C1),.k1(k1),.k2(k2),.sum(DIFF[1]),.carry(C2));
full_adder full_adder3(.a(A[2]),.b(B1[2]),.c(C2),.k1(k1),.k2(k2),.sum(DIFF[2]),.carry(C3));
full_adder full_adder4(.a(A[3]),.b(B1[3]),.c(C3),.k1(k1),.k2(k2),.sum(DIFF[3]),.carry(barrow));

endmodule
//COMPARATOR
module comparator(A,B,k1,k2,altb,aeqb,agtb) ;
input [3:0] A,B;
input k1,k2 ;

output reg altb,aeqb,agtb ;

always @ *
begin
	if(k1==1&&k2==0)
	begin
		altb = A < B ;
		aeqb = A == B ;
		agtb = A > B ;
	end
	else 
	begin
		altb = 0 ;
		aeqb = 0 ;
		agtb = 0 ;
	end
end

endmodule 

