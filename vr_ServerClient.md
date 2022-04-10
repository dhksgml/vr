### VR 머리, 왼쪽 컨트롤러, 오른쪽 컨트롤러 위치, 회전 값을 받아서 서버로 전송

- client.cs

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.Net.Sockets;
using System.Text;
using System.Threading;
using UnityEngine;

public class client : MonoBehaviour
{
	#region private members 	
	private TcpClient socketConnection;
	private Thread clientReceiveThread;
	private Vector3[] pos_obj = new Vector3[3];
	private Vector3[] rot_obj = new Vector3[3];

	private bool isReady = false;
	#endregion
	public GameObject[] body = new GameObject[3];
	//public Transform[] target_Obj = new Transform[3];
	public Transform[] original_Obj = new Transform[3];
	public int array_counter = 0;

	public string serverMessage;
	public string[] result;
	// Use this for initialization 	
	void Start()
	{
		for(int i = 0; i < 3; i++)
        {
			body[i] = Instantiate(body[i]);
        }
		ConnectToTcpServer();
		if(isReady)
			StartCoroutine(CountTime(0.03f));
	}
	IEnumerator CountTime(float delayTime)
	{
		SendMessage();
		yield return new WaitForSecondsRealtime(delayTime);
		StartCoroutine(CountTime(delayTime));
	}
	// Update is called once per frame
	void Update()
	{
		//result = serverMessage.Split(new char[] { ' ' });
		for (int a = 0; a < 3; a++)
        {
			//target_Obj[a].SetPositionAndRotation(pos_obj[a], Quaternion.Euler(rot_obj[a]));
			body[a].transform.SetPositionAndRotation(pos_obj[a], Quaternion.Euler(rot_obj[a]));

        }
		//if (Input.GetKeyDown(KeyCode.Space))
		//{
		//	SendMessage();
		//}
	}
	/// <summary> 	
	/// Setup socket connection. 	
	/// </summary> 	
	private void ConnectToTcpServer()
	{
		try
		{
			clientReceiveThread = new Thread(new ThreadStart(ListenForData));
			clientReceiveThread.IsBackground = true;
			clientReceiveThread.Start();
			isReady = true;
		}
		catch (Exception e)
		{
			Debug.Log("On client connect exception " + e);
		}
	}
	/// <summary> 	
	/// Runs in background clientReceiveThread; Listens for incomming data. 	
	/// </summary>     
	private void ListenForData()
	{
		try
		{
			socketConnection = new TcpClient("127.0.0.1", 4800); //192.168.0.9
			Byte[] bytes = new Byte[1024];
			while (true)
			{
				// Get a stream object for reading 				
				using (NetworkStream stream = socketConnection.GetStream())
				{
					int length;
					// Read incomming stream into byte arrary. 					
					while ((length = stream.Read(bytes, 0, bytes.Length)) != 0)
					{
						var incommingData = new byte[length];
						Array.Copy(bytes, 0, incommingData, 0, length);
						// Convert byte array to string message. 						
						serverMessage = Encoding.ASCII.GetString(incommingData);
						Debug.Log("server message received as: " + serverMessage);
						result = serverMessage.Split(new char[] { ' ' });
						array_counter = 0;
						for (int a = 0; a < 18; a += 6)
                        {
							pos_obj[array_counter] = new Vector3(float.Parse(result[a]), float.Parse(result[a + 1]), float.Parse(result[a + 2]));
							rot_obj[array_counter++] = new Vector3(float.Parse(result[a+3]), float.Parse(result[a + 4]), float.Parse(result[a + 5]));

						}
					}
				}
			}
		}
		catch (SocketException socketException)
		{
			Debug.Log("Socket exception: " + socketException);
		}
	}
	/// <summary> 	
	/// Send message to server using socket connection. 	
	/// </summary> 	
	private void SendMessage()
	{
		if (socketConnection == null)
		{
			return;
		}
		try
		{
			// Get a stream object for writing. 			
			NetworkStream stream = socketConnection.GetStream();
			if (stream.CanWrite)
			{
				string clientMessage = null;
				for (int i= 0; i < 3; i++)
                {
					clientMessage += original_Obj[i].position.x + " " + original_Obj[i].position.y + " " + original_Obj[i].position.z + " " + original_Obj[i].eulerAngles.x + " " + original_Obj[i].eulerAngles.y + " " + original_Obj[i].eulerAngles.z + " ";

				}
				// Convert string message to byte array.                 
				byte[] clientMessageAsByteArray = Encoding.ASCII.GetBytes(clientMessage);
				// Write byte array to socketConnection stream.                 
				stream.Write(clientMessageAsByteArray, 0, clientMessageAsByteArray.Length);
				//Debug.Log(clientMessageAsByteArray);
				//Debug.Log("Client sent his message - should be received by server");
				Debug.Log(clientMessage);
			}
		}
		catch (SocketException socketException)
		{
			Debug.Log("Socket exception: " + socketException);
		}
	}
}
```



- server.cs

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;
using UnityEngine;

public class server : MonoBehaviour
{
	#region private members 	
	/// <summary> 	
	/// TCPListener to listen for incomming TCP connection 	
	/// requests. 	
	/// </summary> 	
	private TcpListener tcpListener;
	/// <summary> 
	/// Background thread for TcpServer workload. 	
	/// </summary> 	
	private Thread tcpListenerThread;
	/// <summary> 	
	/// Create handle to connected tcp client. 	
	/// </summary> 	
	private TcpClient connectedTcpClient;
	#endregion
	public Transform gameobj;
	public string clientMessage = null;

	// Use this for initialization
	void Start()
	{
		// Start TcpServer background thread 		
		tcpListenerThread = new Thread(new ThreadStart(ListenForIncommingRequests));
		tcpListenerThread.IsBackground = true;
		tcpListenerThread.Start();
	}

	// Update is called once per frame
	void Update()
	{
		SendMessage();
    }

	/// <summary> 	
	/// Runs in background TcpServerThread; Handles incomming TcpClient requests 	
	/// </summary> 	
	private void ListenForIncommingRequests()
	{
		try
		{
			// Create listener on localhost port 8052. 			
			tcpListener = new TcpListener(IPAddress.Parse("192.168.0.9"), 4800);
			tcpListener.Start();
			Debug.Log("Server is listening");
			Byte[] bytes = new Byte[1024];
			while (true)
			{
				using (connectedTcpClient = tcpListener.AcceptTcpClient())
				{
					// Get a stream object for reading 					
					using (NetworkStream stream = connectedTcpClient.GetStream())
					{
						int length;
						// Read incomming stream into byte arrary. 						
						while ((length = stream.Read(bytes, 0, bytes.Length)) != 0)
						{
							var incommingData = new byte[length];
							Array.Copy(bytes, 0, incommingData, 0, length);
							// Convert byte array to string message. 							
							clientMessage = Encoding.ASCII.GetString(incommingData);
							//Debug.Log("client message received as: " + clientMessage);
							//string[] result = clientMessage.Split(new char[] { ' ' });
						}
					}
				}
			}
		}
		catch (SocketException socketException)
		{
			Debug.Log("SocketException " + socketException.ToString());
		}
	}
	/// <summary> 	
	/// Send message to client using socket connection. 	
	/// </summary> 	
	private void SendMessage()
	{
		if (connectedTcpClient == null)
		{
			return;
		}

		try
		{
			// Get a stream object for writing. 			
			NetworkStream stream = connectedTcpClient.GetStream();
			if (stream.CanWrite)
			{
				if(clientMessage!= null)
                {
					//Debug.Log("클라이언트 접속 중..");
					string serverMessage = clientMessage;
					byte[] serverMessageAsByteArray = Encoding.ASCII.GetBytes(serverMessage);
					// Write byte array to socketConnection stream.               
					stream.Write(serverMessageAsByteArray, 0, serverMessageAsByteArray.Length);
					clientMessage = null;
				}
				//string serverMessage = gameobj.position.x + " " + gameobj.position.y + " " + gameobj.position.z + " " + gameobj.eulerAngles.x + " " + gameobj.eulerAngles.y + " " + gameobj.eulerAngles.z;
				// Convert string message to byte array.                 
				//byte[] serverMessageAsByteArray = Encoding.ASCII.GetBytes(serverMessage);
				//// Write byte array to socketConnection stream.               
				//stream.Write(serverMessageAsByteArray, 0, serverMessageAsByteArray.Length);
				//Debug.Log("Server sent his message - should be received by client");

			}
		}
		catch (SocketException socketException)
		{
			Debug.Log("Socket exception: " + socketException);
		}
	}
}
```



