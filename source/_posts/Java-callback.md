---
title: Java - callback
date: 2020-02-05 18:57:58
categories: BackEnd
tags:
    - Java
    - Callback
top:
---
In C/ C++, callback refers to the mechanism of calling a function from another function. Memory address of a function is represented as function pointer here. SO the callback is achieved by passing the pointer of func1() to func2().

However, in java, there is no function pointer existing. And we use a callback object or a callback interface, and the interface is passed that refers to the location of a function. 


Below is an example to compute tax by state tax and fedaral tax. Suppose federal tax keeps same while state tax vary by state. We can build interface and implements interface to realize it. 
    // Java program to demonstrate callback mechanism 
    // using interface is Java 
    
    // Create interface 
    import java.util.Scanner; 
    interface STax { 
    	double stateTax(); 
    } 
    
    // Implementation class of Punjab state tax 
    class Punjab implements STax { 
    	public double stateTax() 
    	{ 
    		return 3000.0; 
    	} 
    } 
    
    // Implementation class of Himachal Pardesh state tax 
    class HP implements STax { 
    	public double stateTax() 
    	{ 
    		return 1000.0; 
    	} 
    } 
    
    class TAX { 
    	public static void main(String[] args) 
    throws ClassNotFoundException, IllegalAccessException, InstantiationException 
    	{ 
    		Scanner sc = new Scanner(System.in); 
    		System.out.println("Enter the state name"); 
    		String state = sc.next(); // name of the state 
    
    		// The state name is then stored in an object c 
    		Class c = Class.forName(state); 
    
    		// Create the new object of the class whose name is in c 
    		// Stax interface reference is now referencing that new object 
    		STax ref = (STax)c.newInstance(); 
    
    		/*Call the method to calculate total tax 
    		and pass interface reference - this is callback . 
    		Here, ref may refer to stateTax() of Punjab or HP classes 
    		depending on the class for which the object is created 
    		in the previous step 
    		*/
    
    		calculateTax(ref); 
    	} 
    	static void calculateTax(STax t) 
    	{ 
    		// calculate central tax 
    		double ct = 2000.0; 
    
    		// calculate state tax 
    		double st = t.stateTax(); 
    		double totaltax = st + ct; 
    
    		// display total tax 
    		System.out.println("Total tax =" + totaltax); 
    	} 
    } 
