---
layout: post
title: "Election Service as a Distributed System "
published: true
---

In this post I will implement a service that is remotely invoked by clients using Java RMI.

##The Election Service
The interface of the Election service provides two remote methods:

- vote: This method has two parameters through which the client supplies the name of a candidate (a string) and the “voter's number” (an integer used to ensure each user votes once only). The voter's numbers are allocated sparsely from the range of integers to make them hard to guess.
- result: This method has two parameters through which the server supplies the client with the name of a candidate and the number of votes for that candidate.


{% highlight java %}
public interface election  extends Remote{
	public void vote (String candidate, int voteNum) throws RemoteException;
	public String result() throws RemoteException;
}
{% endhighlight %}


The Election service must record a vote whenever a user thinks they have cast a vote. Also, The records should remain consistent when it is accessed concurrently by multiple clients.


When the client calls the vote method, it has to provide a voter number in the parameter. When the voter number is received by the server, it will check the voter number with an array list of voter numbers. Therefore the server ensures the user only votes once. Java RMI provides call semantics that also helps makes sure a vote is recorded whenever a client sends a call.

Whenever a client casts a vote. The server will save the array of votes into a file. The recovery of the votes after a crash is the server reading from the file. 

At first there was an attempt to code in a mutex system. This mutex system would act as a method of synchronization similar to how process synchronization is handled in operating systems. But, after testing it was seen that mutex isn’t necessary in this program. In the program, a client’s vote is essentially added to a data structure such as an array I have in my code. When I ensured a voter has a recorded vote, it checks with the server to see if the voter number has already been casted. In this, it acts as a synchronization for the server since each client will be checked and the access of array is synchronized already by Java and the OS. 


{% highlight java %}
public class electionImplementation extends UnicastRemoteObject implements election {

	String[] candidates = {"bob","jake"};
	int[] votes = new int[candidates.length];
	ArrayList<Integer> voters = new ArrayList<Integer>();
	private static final long serialVersionUID = 1L;
	protected electionImplementation() throws RemoteException {
		super();
		// TODO Auto-generated constructor stub
	}
	
	public void vote (String candidate, int voteNum)
	{
		
		if (!voters.contains(voteNum))
		{
			for (int i = 0; i < candidates.length; i++)
			{
				if (candidate.equals(candidates[i]))
				{
					votes[i]++;
				}
			}
			try
			{
				File file = new File("backup.txt");
				file.createNewFile();
				PrintWriter output = new PrintWriter(new FileWriter(file));
				for (int i: votes)
				{
					output.println(i);
				}
				output.close();
			}
			catch (Exception e)
			{
				e.printStackTrace();
				System.out.println("No such file exists.");
			}
			voters.add(voteNum);
		}
	}

	public Result result() throws RemoteException {
		// TODO Auto-generated method stub
		Result re = new Result();
		re.setName(candidates);
		re.setVotes(votes);
		return re;
	}
}

{% endhighlight %}

{% highlight java %}
public class electionServer {

	public static void main(String[] args) throws RemoteException {
		// TODO Auto-generated method stub
	    if (System.getSecurityManager() == null) { 
	 	   System.setSecurityManager(new RMISecurityManager()); 
	     }
		try{
			electionImplementation elec = new electionImplementation();
		Naming.rebind("rmi://localhost/election", elec);
		}
		catch(Exception e)
		{
			System.out.println("Exception "+ e.getMessage());
		    e.printStackTrace(); 
		}
	}
}
{% endhighlight %}