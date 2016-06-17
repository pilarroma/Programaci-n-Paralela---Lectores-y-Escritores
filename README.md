*ProgramacionParalela --- Lectores-y-Escritores*

*Programa en Python en el que se plantea una solución al conocido problema de los Lectores y Escritores*

*Utilizacion de Tkinter para crear una interfaz gráfica en la que se ve a los lectores y escritores interactuar*

**--------------------------------------------------------------------------------------------------------------------------------------**

import time

import random 

from multiprocessing import Process

from multiprocessing import Condition,Manager 

from multiprocessing import Queue

from Tkinter import *

k = 10

lec_espera = 0

es_espera = 0

def esc(id,condition,lista,es_espera,lec_espera,queue):


	for v in range(k):
     
     queue.put((id,"piensa",v))		
     print 'Escritor piensa en lo que vas a escribir',v
     condition.acquire()
     while lista[1]==1 and lista[0]>0:
	    queue.put((id,"espera",v))
	    print "Escritor espera"
	    time.sleep(random.random()/2)
	    es_espera += 1
	    condition.wait()
	    
     lista[0] = lista[0]+1 #numero escritores
     es_espera -= 1
    
     condition.release()	
     queue.put((id,"escribe",v))					
     print "Escritor escribe",v
      
     lec_espera += 1    
     time.sleep(random.random()/2)
     lec_espera -= 1
      
     condition.acquire()
     lista[0] = lista[0]-1
     
     condition.notify_all()
     condition.release() 	 	
     
def lec(id,condition,lista,es_espera,lec_espera,queue):


	for v in range(k):
     
      condition.acquire()
    
      while lista[0] > 0 :
	    queue.put((id,"espera",v))
      	    print "lector esperando"
     	    time.sleep(random.random()/2)
     	    lec_espera += 1
     	    condition.wait()
      
      lista[1] = lista[1] + 1 #numero de lectores
      lec_espera -= 1
      
      condition.release()
      queue.put((id,"lee",v))	                 
      print "Lector lee",v 
     
      time.sleep(random.random()/2)
        
      condition.acquire()
      lista[1] = lista[1] - 1
      
      condition.notify_all()
      condition.release()
      
      queue.put((id,"reflexiona",v))	
      print "Lector reflexiona sobre lo leido", v

if __name__=="__main__":
	
	root = Tk()
	root.title("Una biblioteca de prueba")	
	
	frame = Frame(root)
	frame.pack()
	
	w = 600
	h = 500

	canvas = Canvas(frame, width= w, height= h, bg="orange")
	canvas.grid(row=0, column=0, columnspan= 5)
	
	obj = []
	
	NLectores = 4
	NEscritores = 6
  color = ["red","green","blue","grey","yellow","black"]
	
	obj.append(canvas.create_text(280,20,text = "BIBLIOTECA"))
	obj.append(canvas.create_rectangle(180,40,400,300,fill = "white"))
	
	obj.append(canvas.create_text(40,20,text = "ESCRITORES"))
	
	for i in range(NEscritores):
	  obj.append(canvas.create_oval(40,40+i*60,80,80+i*60,fill = color[i%6]))

	obj.append(canvas.create_text(500,20,text = "LECTORES"))

	for i in range(NLectores):
	  obj.append(canvas.create_rectangle(500,40+i*50,540,80+i*50,fill = color[i%6])) 	
			
	queue = Queue()

	condition = Condition()
	manager = Manager()
	lista = manager.list([0,0])

	def ejec_start():	
	  listae=["escritor1","escritor2","escritor3"]
	  listal=["lector1","lector2"]
 
	  listaescritores = [] 
	  listalectores = []
 	
	  global escritor
	  for i in range(len(listae)):
	   escritor = Process(target = esc,name = listae[i],args = (1,condition,lista,es_espera,lec_espera,queue))	
	   listaescritores.append(escritor)
	
	  global lector
	  for i in range(len(listal)):
	   lector = Process(target = lec,name = listal[i],args = (2,condition,lista,es_espera,lec_espera,queue))		
	   listalectores.append(lector)                                      
	   
  	for escritor in listaescritores:
	    escritor.start()
	  for lector in listalectores:
	    lector.start() 
	  
    for escritor in listaescritores:
	    escritor.join()
	  for lector in listalectores:
	    lector.join()
	
	# Disenyo del boton start    
	start = Button(frame, text = "Start", command = ejec_start)
	start.grid(row = 1,column = 2)

	def ejec_end():
	  queue.put("quit")

  # Disenyo del boton end
	end = Button(frame, text = "End", command = ejec_end)
	end.grid(row = 2,column = 2)
	
	etiqueta1 = Label(frame, text = "Numero de iteraciones")
	etiqueta1.grid(row = 1,column = 0)
	
	nk = StringVar()
	nk.set(str(10))
        
	entrada1 = Entry(frame, width = 2, textvariable = nk)
	entrada1.grid(row = 1,column = 1) 
	
	try:
	  while True:
	    if not queue.empty():
		    s = queue.get()
		    if s == "quit":
		      break
	      else:
		      if s[0]==1:
		        if s[1] == "piensa":
		          canvas.itemconfigure(s[0],fill = "orange")
		          canvas.itemconfigure(3,text = "piensa"+str(s[2]))
		          time.sleep(random.random()/2)	
		        elif s[1] == "espera":
    		      canvas.itemconfigure(s[0],fill = "yellow")
		          canvas.itemconfigure(3,text = "espera"+str(s[2]))
	            time.sleep(random.random()/2)
		        elif s[1] == "escribe":
		          canvas.itemconfigure(s[0],fill = "blue")
		          canvas.itemconfigure(3,text = "escribe"+str(s[2]))
	            time.sleep(random.random()/2)	
		      elif s[0]==2:
		        if s[1] == "espera":
		          canvas.itemconfigure(s[0],fill = "purple")
		          canvas.itemconfigure(4,text = "espera"+str(s[2]))
		          time.sleep(random.random()/2)	 
		        elif s[1] == "lee":
		          canvas.itemconfigure(s[0],fill = "red")
		          canvas.itemconfigure(4, text = "lee"+str(s[2]))
		          time.sleep(random.random()/2)				
		        elif s[1] == "reflexiona":	
		          canvas.itemconfigure(s[0],fill = "pink")
		          canvas.itemconfigure(4, text = "reflexiona"+str(s[2]))
		          time.sleep(random.random()/2)
	    root.update()
	except TclError:
		print "alguna excepcion"
	print "fin"
