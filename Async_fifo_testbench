// Testbench //
module FIFO_tb();
    parameter DSIZE = 8;               // Data bus size
    parameter ASIZE = 3;               // Address bus size
    parameter DEPTH = 1 << ASIZE;      // Depth of the FIFO memory
    
    reg  [DSIZE-1:0] wdata;            // Input data
    wire [DSIZE-1:0] rdata;            // Output data
    wire wfull, rempty;                // Full and empty flags
    reg  winc, rinc, wclk, rclk;
    reg  wrst_n, rrst_n;
    
    // Instantiate DUT
    FIFO #(DSIZE, ASIZE) fifo (
        .rdata(rdata), 
        .wdata(wdata),
        .wfull(wfull),
        .rempty(rempty),
        .winc(winc), 
        .rinc(rinc), 
        .wclk(wclk), 
        .rclk(rclk), 
        .wrst_n(wrst_n), 
        .rrst_n(rrst_n)
    );
    
    integer i;
    
    // Clock generation
    always #5  wclk = ~wclk;    // Write clock (100 MHz)
    always #10 rclk = ~rclk;    // Read clock  (50 MHz)
    
    initial begin
        // Initialize all signals
        wclk = 0;
        rclk = 0;
        wrst_n = 1;
        rrst_n = 1;
        winc = 0;
        rinc = 0;
        wdata = 0;
        i = 0;
        
        // Reset the FIFO
        #40 wrst_n = 0; rrst_n = 0;
        #40 wrst_n = 1; rrst_n = 1;
        $display("[%0t] Reset Released", $time);
        
      
        // TEST CASE 1: Write data and read it back

        $display("[%0t] TEST 1: Write & Read Back ", $time);
        rinc = 1;
        for (i = 0; i < 10; i = i + 1) begin
            @(posedge wclk);
            wdata = $random % 256;
            winc = 1;
            $display("[%0t] WRITE :%h", $time, wdata);
            @(posedge wclk);
            winc = 0;
        end
        

        // TEST CASE 2: Fill FIFO and attempt extra writes

        $display("[%0t] TEST 2: Fill FIFO ", $time);
        rinc = 0;
        winc = 1;
        for (i = 0; i < DEPTH + 3; i = i + 1) begin
            @(posedge wclk);
            wdata = $random % 256;
            $display("[%0t] WRITE attempt: %h (Full=%b)", $time, wdata, wfull);
        end
        winc = 0;
        
        // TEST CASE 3: Read from empty FIFO (underflow)
        
        $display("[%0t]  TEST 3: Read Empty FIFO ", $time);
        winc = 0;
        rinc = 1;
        for (i = 0; i < DEPTH + 3; i = i + 1) begin
            @(posedge rclk);
            $display("[%0t] READ attempt: 0x%0h (Empty=%b)", $time, rdata, rempty);
        end
        rinc = 0;
        
        // Finish simulation
        #50;
        $display("[%0t] Simulation Complete ", $time);
        $finish;
    end
    
    initial begin
        $dumpfile("fifo_tb.vcd");   // VCD output file name
        $dumpvars(0, FIFO_tb);      
    end
endmodule
