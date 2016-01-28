# -*- coding: cp1252 -*-
import urllib2
from bs4 import BeautifulSoup
from Tkinter import *


class deal():
    def __init__(self, name, price, time, lowby, ship):
        self.name = name
        self.price = price
        self.time = time
        self.lowby = lowby
        self.ship = ship
    def evaluate(self):
        val_lowby = float(E1.get())
        print(val_lowby)
        percent = float(var.get())
        if ',' in self.lowby:
            lowby1 = (self.lowby.replace(",", ""))
        else:
            lowby1 = self.lowby

        if '$' in lowby1:
            try:
                lowby1 = float(lowby1.replace("$", ""))
                if val_lowby <=  lowby1:
                    return self
            except:
                print('Could not evaluate',self.name)
                return
            
            
        elif '%' in self.price:
             try:
                 percent1 = float(self.price.replace("%", ""))
             except:
                print('Could not evaluate',self.name)
                return
            
             if percent1 >= percent:
                 return self
             else:
                 #print(self.name, 'not A DEAL', self.price)
                 return
       
        #else:
        #    print(self.name, 'not A DEAL',self.lowby)
            

        

        
        
        
        
def clean_up(aLink):
    html = urllib2.urlopen(aLink).read()
    soup = BeautifulSoup(html,"html5lib")
    texts = soup.findAll(text=True)
    [s.extract() for s in soup(['style', 'script', '[document]', 'head', 'title'])]
    visible_text = soup.getText().encode('utf-8').strip()
    l = visible_text.split('\n')
    l = filter(lambda element: (('â' or '€') not in element) and (element != '') and ('\xc2\xa0' not in element), l)
   
    #Cleaning up the crawled data
    for i in range(len(l)):
        if 'for discount' in l[i]:
            deals_start = i + 1

    clean_list = []
    for i in range(deals_start, len(l)):
        
        if 'COMMENT' in l[i]:
            l[i] = '************break***************'
        elif (l[i].isupper()):
            continue
        elif 'miss a single chance' in l[i]:
            break
       
      
        clean_list.append(l[i])

       
    return clean_list



def organize(clean_list):
     pos = 0
     deals_list = []
     while pos < len(clean_list):
         body_text = ''
         name = clean_list[pos]
         pos+=1
         price_shipping = clean_list[pos]
         pos+=2
         time = clean_list[pos]
         pos+=1
         while '**break**' not in clean_list[pos]:
             body_text+= clean_list[pos]
             pos+=1
         l = price_shipping.split('  ')
         l2 = []
         for element in l:
             if len(element) > 1:
                 l2.append(element)
         price_raw = l2[0]
         ship = l2[1]
         
         if '$' in price_raw:
             price = (price_raw.replace("$", ""))
         elif '%' in price_raw:
             price = 0
             percents = []
             pricelist = price_raw.split(' ')
             for word in pricelist:
                 if '%' in word:
                     percents.append(word.replace("%", ""))
             for val in percents:
                 if int(val) > price:
                     price = int(val)
             price = str(price) + '%'
                 
        
                   
         #Calculate the time since posting by converting hours into minutes, puts time in minutes into the time variable
         time_list = time.split(' ')
         minutes = 0
         for i in range(len(time_list)):
            if time_list[i] == 'hr':
                minutes += int(time_list[i-1]) * 60
            elif time_list[i] == 'min':
                minutes += int(time_list[i-1])
         time = minutes

         #Find the dollars off the deal is
         lowby = '0'
         keywords1 = ['findby','dealby','savingsof']
         keywords2 = ['belowlist']
         keywords3 = ['off']
         if '%' not in price:                     
             body_list = body_text.split(' ')
             for i in xrange(len(body_list)-1):

                oneword = body_list[i]
                twoword = body_list[i] + body_list[i+1]
                if oneword in keywords3:
                    lowby = body_list[i-1]
                elif twoword in keywords1:
                    lowby = body_list[i+2]
                elif twoword in keywords2:
                    lowby = body_list[i-1]
                    
                
                   
                
 
         
         
         item = deal(name,price,time,lowby,ship)
         deals_list.append(item)
         pos+=1    
     return deals_list

def checkDeals(dealsList):
    finalDeals = []
    for item in dealsList:
        print(item.name)
        print(item.price)
        print(item.lowby)
        check = item.evaluate() 
        if check != None:
            finalDeals.append(check)
    return finalDeals
        
    

def driverFunction():
    finalList = checkDeals(organize(clean_up('http://dealnews.com/c202/Clothing-Accessories/f1/Mens/?o=time')))
   

    for i in range(len(finalList)):
       Label(frame,text=finalList[i].name).grid(row=i,column=0)
       if '%' in finalList[i].price:
            Label(frame,text="Clearance: "+finalList[i].price, justify = LEFT).grid(row=i,column=1)
       else:
            Label(frame,text="Price: "+finalList[i].price, justify = LEFT).grid(row=i,column=1)
            Label(frame,text="Low by: " + finalList[i].lowby, justify = LEFT).grid(row=i,column=2)

        
def myfunction(event):
    canvas.configure(scrollregion=canvas.bbox("all"),width=500,height=200)
   


top = Tk()
sizex = 400
sizey = 500
posx  = 100
posy  = 100
top.wm_geometry("%dx%d+%d+%d" % (sizex, sizey, posx, posy))
frame1 = LabelFrame(top, text="Low by value ($):")
frame1.pack(fill="both", expand="yes")
E1 = Entry(frame1, bd =2, width = 3)
E1.pack(anchor = N)



frame2 = LabelFrame(top, text="Clearance")
frame2.pack(fill="both", expand="yes")
 
var = IntVar()
R1 = Radiobutton(frame2, text="over 20%", variable=var, value=20)
R1.pack()

R2 = Radiobutton(frame2, text="over 40%", variable=var, value=40)
R2.pack()

R3 = Radiobutton(frame2, text="over 60%", variable=var, value=60)
R3.pack()

B = Button(top, text ="Start", command = driverFunction)
B.pack(side = BOTTOM, anchor = CENTER)

myFrame = Frame(top, bd = 1, width = 500, height = 100)
myFrame.pack(fill="both", expand="yes")

canvas=Canvas(myFrame)
frame=Frame(canvas, width = 600, height = 180)
myscrollbar=Scrollbar(myFrame,orient="vertical",command=canvas.yview)
canvas.configure(yscrollcommand=myscrollbar.set)

myscrollbar.pack(side="right",fill="y")
canvas.pack(side="left")
canvas.create_window((0,0),window=frame,anchor='nw')
frame.bind("<Configure>",myfunction)










top.mainloop()
        

