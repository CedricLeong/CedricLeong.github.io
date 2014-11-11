---
layout: post
title: Election Service with Java RMI
---

In this post I will implement a service that is remotely invoked by clients using Java RMI.

##The Election Service
The interface of the Election service provides two remote methods:
• vote: This method has two parameters through which the client supplies the name of a
candidate (a string) and the “voter's number” (an integer used to ensure each user votes
once only). The voter's numbers are allocated sparsely from the range of integers to
make them hard to guess.
• result: This method has two parameters through which the server supplies the client with
the name of a candidate and the number of votes for that candidate.

The Election service must record a vote whenever a user thinks they have cast a vote. The
records should remain consistent when it is accessed concurrently by multiple clients.

{% highlight java %}
public interface election  extends Remote{
	public void vote (String candidate, int voteNum) throws RemoteException;
	public String result() throws RemoteException;
}
{% endhighlight %}

{% highlight java %}
public class electionImplementation extends UnicastRemoteObject implements election{

	String[] candidates = {"bob","jake"};
	int[] votes = new int[candidates.length];
	private static final long serialVersionUID = 1L;
	protected electionImplementation() throws RemoteException {
		super();
		// TODO Auto-generated constructor stub
	}
	
	public void vote (String candidate, int voteNum)
	{
		for (int i = 0; i < candidates.length; i++)
		{
			if (candidate.equals(candidates[i]))
			{
				votes[i]++;
			}
		}
		
	}
	public String result () throws RemoteException
	{
		return candidates + votes.toString();
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
			electionImplementation lol = new electionImplementation();
		Naming.rebind("rmi://localhost/election", lol);
		}
		catch(Exception e)
		{
			System.out.println("Exception "+ e.getMessage());
		    e.printStackTrace(); 

		}
		
	}

}
{% endhighlight %}

