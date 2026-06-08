Deadlocks

```java
public static void main(String[] args)
{
	Integer lock1=1;
	Integer lock2=2;

	Thread th1=new Thread(()->{
		synchronized(lock1)
		{
			System.out.println("Acquired lock1");
			try
			{
				Thread.sleep(100);
			}
			catch(InterruptedException e){}
			synchronized(lock2)
			{
				System.out.println("My name is: "+Thread.currentThread().getName());
			}
		}
	},"thread1");

	Thread th2=new Thread(()->{
		synchronized(lock2)
		{
			System.out.println("Acquired lock2");
			try
			{
				Thread.sleep(100);
			}
			catch(InterruptedException e){}

			synchronized(lock1)
			{
				System.out.println("My name is: "+Thread.currentThread().getName());
			}
		}
	},"thread2");    

	th1.start();
	th2.start();
}
```

Message Wall 

```Java

class MessageWall
{
    private String msg;
    private volatile boolean isWriting=false;


    MessageWall(){}

    public String readMsg()
    {
        while(isWriting)
        {
            System.out.println("Waiting for an update...");
            try
            {
                Thread.sleep(100);
            }
            catch(InterruptedException e){}
        }

        return msg;
    }

    public synchronized void writeMsg(String msg)
    {
        isWriting=true;
        this.msg=msg;

        try
        {
            Thread.sleep(1000);
        }
        catch(InterruptedException e){}

        isWriting=false;
    }
}

class Writer implements Runnable
{
    String msg;
    MessageWall msgWall;

    Writer(String msg,MessageWall msgWall)
    {
        this.msg=msg;
        this.msgWall=msgWall;
    }

    public void run()
    {
        msgWall.writeMsg(msg);
    }
}

class Reader implements Runnable
{

    MessageWall messageWall;

    Reader(MessageWall msgWall)
    {
        this.messageWall=msgWall;
    }

    public void run()
    {
        System.out.println("Read message by Reader: "+messageWall.readMsg());
        
    }
}

public class message {
    public static void main(String[] args)
    {
        MessageWall wall=new MessageWall();

        Thread reader1=new Thread(new Reader(wall));
        Thread reader2=new Thread(new Reader(wall));

        Thread writer1=new Thread(new Writer("hello everynyan",wall));

        reader1.start();
        reader2.start();
        writer1.start();
    }
}
```

Stack

```Java

class Modifier implements Runnable
{
    protected Stack<Integer> stack;
    int capacity;

    Modifier(Stack<Integer> stack,int cap)
    {
        this.stack=stack;
        capacity=cap;
    }

    public void run()
    {

    }
}

class Pusher extends Modifier
{
    int item;
    Pusher(Stack<Integer> stack,int cap,int item)
    {
        super(stack,cap);
        this.item=item;
    }

    @Override
    public void run()
    {
        synchronized(stack)
        {
            while(stack.size()>=capacity)
            {
                System.out.println("Stack is full. Waiting...");
                try{
                    stack.wait();
                }
                catch(InterruptedException e){}
            }

            stack.push(item);
            stack.notifyAll();
        }
    }
}

class Popper extends Modifier
{
    Popper(Stack<Integer> stack,int cap)
    {
        
        super(stack,cap);
    }

    @Override
    public void run()
    {
        synchronized(stack)
        {
            while(stack.isEmpty())
            {
                System.out.println("Stack is empty. Waiting... ");
                try{
                    stack.wait();
                }
                catch(InterruptedException e){}
            }

            stack.pop();
            stack.notifyAll();
        }
    }
}

```

