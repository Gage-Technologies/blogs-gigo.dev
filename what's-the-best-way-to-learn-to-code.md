4-23-2024

# What's the best way to learn to code

![cool AI image](https://raw.githubusercontent.com/Gage-Technologies/blogs-gigo.dev/master/images/codecuhPNG.png)

It’s a question many people ask, but there aren’t many good answers. Typically the discussion shifts to universities or bootcamps with the occasional course on Udemy or Coursera.

My objective today is to outline the skills you must hone to learn code anywhere and hopefully by the end of this you can learn code in any format you choose.

## Survival Skills in Computer Science

Easily the most important skill a programmer can have is Googling. I know it’s a common joke but it’s true, but to make it sound a bit more professional let’s refer to it as “researching”. Now you can’t simply type your problem into a search bar and always get the answer, and this is where the actual skill of research comes in.

Understanding your problem to the best of your ability and using forums like StackOverflow are extremely helpful for diagnosing and assessing problems.

This allows programmers to not only have the possibility of fixing their problem but gives them more context on similar problems others have. Chances are you found this article from google so it’s already working!

## Organization and Architecture

### Architecture

It’s important to build a strong foundation when learning programming, starting with the fundamentals can help with this. Finding a course (or even simply reading wikipedia) on the structure of a computer will enlighten you about many of the early decisions made for programming languages.

A good example of how this understanding can help you is with arrays in compiled languages in C++. The array is a pointer to a memory address and can be manipulated similar to any other pointer. Strings in C++ can also be constructed with arrays of chars which makes sense if you have a good foundation from computer architecture.

### Organization

How code is organized is typically what differentiates a novice from a veteran programmer. It’s a crucial structure that all programmers agree upon although some may deviate.

However, once you familiarize yourself with this structure, reading and understanding code becomes much more simple.

An example of conforming to this structure can be seen in the code below:

public class Calculator {

    // Adds two numbers
    public double add(double number1, double number2) {
        return number1 + number2;
    }

    // Subtracts the second number from the first
    public double subtract(double number1, double number2) {
        return number1 - number2;
    }

    // Multiplies two numbers
    public double multiply(double number1, double number2) {
        return number1 * number2;
    }

    // Divides the first number by the second
    public double divide(double number1, double number2) {
        if (number2 == 0) {
            throw new IllegalArgumentException("Cannot divide 
            by zero.");
        }
        return number1 / number2;
    }

    // Main method to test the Calculator class
    public static void main(String[] args) {
        Calculator calc = new Calculator();

        // Test the methods
        System.out.println("5 + 3 = " + calc.add(5, 3));
        System.out.println("10 - 2 = " + calc.subtract(10, 
        2));
        System.out.println("4 * 7 = " + calc.multiply(4, 7));
        System.out.println("20 / 5 = " + calc.divide(20, 5));
        // Uncomment the following line to test division by 
        // zero
        // System.out.println("10 / 0 = " + calc.divide(10, 
        // 0));
    }
}


While you may not be an experienced programmer, it is still very clear what this program is doing based on how it is structured. Compare that file with this one:

      public class Calculator{
      public double add(double number1,double number2){return number1+number2;}
      public double subtract(double number1, double number2){return number1-number2;}
      public double multiply(double number1, double number2)
      {
      return number1*number2;
      }
      public double divide(double number1, double number2){if (number2 == 0) {throw new IllegalArgumentException("Cannot divide by zero.");} return number1 / number2;}
      public static void main(String[] args){Calculator calc=new Calculator();System.out.println("5 + 3 = " + calc.add(5, 3));System.out.println("10 - 2 = " + calc.subtract(10, 2));System.out.println("4 * 7 = " + calc.multiply(4, 7));System.out.println("20 / 5 = " + calc.divide(20, 5));
      // Uncomment the following line to test division by zero
    // System.out.println("10 / 0 = " + calc.divide(10, 0));}}

As you can see this code is very hard to read. While you may be able to understand the general idea of what this program is doing, it is very hard to pinpoint what exactly one line of code is doing.

This is a skill that can be easily picked up by examining other programmers' code. A great resource for learning this quickly is to look at open source repositories on Github. Once you have seen good program structure enough you will unknowingly begin to structure your code similarly.

## What’s the best language to start with?

Many beginners believe this to be the most important question that needs to be answered before starting to learn programming. This can be the case depending on why you are learning to code, but generally speaking the first language you learn is not really that important.

There are some languages that are easier to pick up than others, Python is a good example of a beginner friendly language. However, your first language is typically only important in that you should enjoy using it.

For the vast majority of programmers, they don’t simply stick to one language, they learn multiple languages as each language has its own advantages and disadvantages.

## The Advantages of Being Poly-Lingual

Once you start learning your first language, it’s a good idea to learn some basics of another language. Programming languages share much in common with each other similar to human language.

If you learn a sentence in multiple languages it cements the commonalities between the languages quicker. With each language you learn you can find the quirks and recognize the use cases with each language which is easily the biggest weakness of new programmers.

If I had to give a recommendation on what languages to learn, I would say start with Python and then learn Golang. Python is known for being beginner friendly, however, due to the fact that Python is loosely typed many beginners that learn only python do not get used to declaring and dealing with types. Golang is a strongly typed language and has many conventions that are similar to languages like Java, C++, and Rust.

## Conclusion

If you are looking for a more concrete suggestion on where to learn code, [GIGO Dev](gigo.dev) offers the most varied platform for learning code.

The most important advice for learning code, wherever you end up doing it, is to go at your own rate. Learn what you want to learn to accomplish tasks you care to finish.

[This article](https://dev.to/gigo_dev/whats-the-best-way-to-learn-to-code-27f2) originally posted on [dev.to](dev.to) 

