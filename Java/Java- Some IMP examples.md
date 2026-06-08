```Java

package Collections;

// 1. implement Comparable
public class Student implements Comparable<Student>{

    static int ID=0;

    String name;
    int roll;
    double score;

    Student(String name,double score)
    {
        this.name=name;
        this.roll=ID++;
        this.score=score;
    }

    double getScore()
    {
        return score;
    }

    // 2. write compareTo
    @Override
    public int compareTo(Student s)
    {
        /*
        this<s -> return -1 (or any -ve number)
        this>s -> return 1 (or any +ve number)
        this==s -> return 0 

        actually if you want to sort 'this' before s then return -1
        else return 1 or 0
        */

        // asecnding
        // if(this.score<s.score) return -1;
        // if(this.score>s.score) return 1;
        // return 0;

        // descending
        if(this.score>s.score) return -1;
        if(this.score<s.score) return 1;
        return 0;
    }

    public String toString()
    {
        return name;
    }
}

package Collections;

import java.util.Comparator;
import java.util.PriorityQueue;

// 1. implement Comparator<T>
class MyComparator implements Comparator<Integer>
{
    // 2. override the compare(T o1,T o2) method
    @Override
    public int compare(Integer a,Integer b)
    {
        return b-a;// sort descending
    }
}

public class comparator {

    public static void main(String[] args)
    {
        PriorityQueue<Student> pq=new PriorityQueue<>();// this wouldn't work without Comparable

        pq.offer(new Student("Niggs", 34.2));
        pq.offer(new Student("Tom",67));
        pq.offer(new Student("Romit", 21));

        while(!pq.isEmpty())
        {
            Student s=pq.poll();
            System.out.println(s);
        }

        // but what if we wanted to change behaviour of priority for Integer? 

        PriorityQueue<Integer> pq1=new PriorityQueue<>(new MyComparator());// PriorityQueue takes in an object of Comparator interface

        // difference?
        // If a class implements an ordering (thru Comparable) --> Natural Ordering of the class
        // If a comparator object is passed (thru Comparator) --> Total ordering (used to override Natural ordering)



        // using lambdas
        PriorityQueue<Student> pq2=new PriorityQueue<>( (a,b) -> {
            if(a.getScore()>b.getScore()) return -1;
            if(a.getScore()<b.getScore()) return 1;
            return 0;
        } );

        PriorityQueue<Integer> pq3=new PriorityQueue<>((a,b)->b-a);
    }
}


```

Iterators
```Java

package Collections;
import java.util.Iterator;


class GenericList<T> implements Iterable<T>
{
    private T[] items;
    private int size;

    public GenericList(int size)
    {
        this.size=size;
        items=(T[])new Object[size];
    }

    public void add(T x)
    {
        items[size++]=x;
    }

    public T get(int i)
    {
        return items[i];
    }

    // 2. define an iterator() method, this will create and send an iterator object to this Class
    @Override
    public Iterator<T> iterator()
    {
        return new MyIterator(this);
    }

    // 1. define a private nested iterator class which implements Iterator 
    private class MyIterator implements Iterator<T>
    {

        // 1.0 define the object that iterator iterates through
        
        private GenericList<T> list;
        private int index=0;

        MyIterator(GenericList<T> list)
        {
            this.list=list;
        }

        // 1.1 define hasNext() and next()
        @Override
        public boolean hasNext()
        {
            return index<list.size;
        }

        @Override
        public T next()
        {
            if(!hasNext()) return null;
            return list.get(index++);
        }
    }
}

public class generics {
    public static void main(String[] args)
    {
        GenericList<Integer> myarr=new GenericList<>(10);

        // WAY 1
        Iterator<Integer> it=myarr.iterator();

        while(it.hasNext())
        {
            System.out.println(it.next());
        }



        // WAY 2
        for(int x:myarr) System.out.println(x);
    }
}

```

File Handling

```Java
import java.io.BufferedReader;
import java.io.FileReader;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

public class file1 {
    public static void main(String[] args) throws Exception
    {
        Map<String,ArrayList<String>> groups=new HashMap<>();

        try(BufferedReader fw=new BufferedReader(new FileReader("words.txt")))
        {
            String line;
            line=fw.readLine();

            String[] words=line.split(" ");

            for(String word:words)
            {
                char[] chars=word.toCharArray();
                Arrays.sort(chars);

                String s=new String(chars);

                if(!groups.containsKey(s)) groups.put(s, new ArrayList<>());
                
                groups.get(s).add(word);
            }
        }

        for(Map.Entry<String,ArrayList<String>> gr:groups.entrySet())
        {
            System.out.println(gr.getValue());
        }
    }
}
```

```Java
import java.io.DataInputStream;
import java.io.FileInputStream;

public class file2 {
    public static void main(String[] args) {
        
        int mx=0;
        try(DataInputStream dos=new DataInputStream(new FileInputStream("marks.bin")))
        {
            while(true)
            {
                try
                {
                    float marks=dos.readFloat();
                    mx=Math.max(mx,marks);
                }
                catch(EOFException e)
                {
                    break;
                }
            }
        }

        System.out.println("Max marks: ",mx);
    }
}
```

```Java

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.ArrayList;

class Student implements Serializable
{
    int roll;
    String name;
    float score;

    Student(int roll,String name,float score)
    {
        this.roll=roll;
        this.name=name;
        this.score=score;
    }

    void display()
    {
        System.out.println("Name: "+name);
        System.out.println("Roll: "+roll);
        System.out.println("Score: "+score);
    }
}

class StudentManager 
{
    ArrayList<Student> students;
    static int rollGen=0;

    StudentManager()
    {
        students=new ArrayList<>();
    }

    void createStudent(String name,float score)
    {
        Student s=new Student(rollGen++,name,score);
        students.add(s);
    }

    void saveStudentData(String filename) throws IOException
    {
        File saveFile=new File(filename);
        if(!saveFile.exists()) saveFile.createNewFile();

        try(ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream(saveFile,true)))
        {
            for(Student s:students) oos.writeObject(s);
        }
    }

    void displayAllInFile(String filename) throws IOException
    {
        try(ObjectInputStream ois=new ObjectInputStream(new FileInputStream(filename)))
        {
            while(true)
            {
                try
                {
                    Student s=(Student)ois.readObject();
                    s.display();
                }
                catch(EOFException e)
                {
                    break;
                }
            }
        }
    }
}
```

