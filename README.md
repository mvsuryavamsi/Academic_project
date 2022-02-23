# Academic_project
#Face detection for attendance and authorization
import face_recognition 
import cv2
import mysql.connector
import pandas as pd
import pyttsx3
from PIL import Image
from openpyxl import load_workbook
import matplotlib.pyplot as plt
from tabulate import tabulate
from datetime import datetime,timedelta
import os 


mydb = mysql.connector.connect(
host = "localhost",
user="vamsi",
password="V@msi495",
database ="python"
)



#for speech perpose
engine = pyttsx3.init()
voices = engine.getProperty('voices')
engine.setProperty('voice', voices[1].id)
engine.setProperty('rate', 160)
engine.setProperty('volume',1.0)

# capturing image from web cam and verifying the image....
video_capture =cv2.VideoCapture(0)
vamsi_image = face_recognition.load_image_file(r"C:\\vamsi.jpeg")
vamsi_face_encoding = face_recognition.face_encodings(vamsi_image)[0]


vinitha_image = face_recognition.load_image_file(r"C:\Users\mvsur\Downloads\vinitha2.jpeg")
vinitha_face_encoding = face_recognition.face_encodings(vinitha_image)[0]

chathurya_image = face_recognition.load_image_file(r"C:\Users\mvsur\Downloads\chaturya.jpeg")
chathurya_face_encoding = face_recognition.face_encodings(chathurya_image)[0]

nook_image = face_recognition.load_image_file(r"C:\nooks.jpeg")
nook_face_encoding = face_recognition.face_encodings(nook_image)[0]

chaitu_image = face_recognition.load_image_file(r"C:\476.jpeg")
chaitu_face_encoding = face_recognition.face_encodings(chaitu_image)[0]



# Create arrays of known face encodings and their names
pics=[
    vamsi_image,
    vinitha_image,
    chathurya_image,
    nook_image,
    chaitu_image
]
known_face_encodings = [
    vamsi_face_encoding,
    vinitha_face_encoding,
    chathurya_face_encoding,
    nook_face_encoding,
    chaitu_face_encoding
]
known_face_names = [
    "M.vamsi",
    "M.vinitha",
    "chathurya",
    "N.nookaraj",
    "D.chaitanya"

]

    

def details():
    while True:
        ret, frame = video_capture.read()
        face_locations = face_recognition.face_locations(frame)
        face_encodings = face_recognition.face_encodings(frame, face_locations)
        name = "unknown"
        for (top, right,bottom,left),face_encodings in zip(face_locations, face_encodings):
            matches = face_recognition.compare_faces(known_face_encodings, face_encodings)


            if True in matches:
                first_match_index = matches.index(True)
                name = known_face_names[first_match_index]
                pic = pics[first_match_index]
            cv2.rectangle(frame,(left, bottom - 35), (right, bottom),(0,255,0), cv2.FILLED)
            font= cv2.FONT_HERSHEY_DUPLEX
            cv2.putText(frame,name, (left +6,bottom -6), font, 1.0,(255,255,255),1)
        cv2.imshow('recogniting',frame )

        if cv2.waitKey(1) & 0xFF ==   ord('q'):
            break

        if name in known_face_names: 
            #from here getting data from the sql data base
            #print(name)
            mycursor = mydb.cursor()
            mycursor.execute("select * from  parent natural join child where name ="+"\'"+name+"\'")
            myresult = mycursor.fetchall()
            field = [i[0] for i in mycursor.description]
            df1=pd.DataFrame(myresult,columns=field)
            return name,df1,pic
            
        else:
            engine.say('unkown person, data not found in data base')
            engine.runAndWait() 
            return details()
            
        
def display_details():
        name,df1,pic,=details()
        plt.imshow(pic)
        b=df1
        engine.say('Hi'+name+'here is your details')
        
        k=[]
        l=[]
        for i in b:
            k.append(i)
            for j in df1[i].values:
                l.append(j)
        dict = {'Name':k,'values':l}
        df = pd.DataFrame(dict)
        
   

        # displaying the DataFrame
        print(tabulate(df,tablefmt = 'fancy_grid'))
        
        
        
        #break
def attendance():
    name,df2,pic=details()
    file_dt=datetime.strftime(datetime.now()+timedelta(0),'%d-%m-%Y')
    login_time=datetime.now().strftime("%H:%M:%S")
    for i in df2.id.values:
        a=i
        for j in df2.Name.values:
            b=j
    pat=r'C:\Users\mvsur\Documents\final_year_attendance.xlsx'
    wb= load_workbook(pat)
    t=[a,b,login_time]

    if file_dt in wb.sheetnames:
        ws=wb[file_dt]
        ws.append(t)   
        engine.say('Hello'+df2.Name.values+'welcome,have a nice day')
        print("{} is present".format(name))
    else:
        wb.create_sheet(file_dt)
        ws=wb[file_dt]
        
        ws.append(t)   
        engine.say('Hello'+df2.Name.values+'welcome,have a nice day')
        print("{} is present".format(name))
        

   
    
    wb.save(pat)
    #return attendance()

        
   
 
    #return details()
    
n = int(input(''' 
            choose option:

                        1.GET PERSON DATA
                        2.ATTENDANCE
            
            '''))
if n==1:
    display_details()
elif n== 2:
    attendance()
       
   
    
    
       
    
    


engine.runAndWait()    
video_capture.release()
cv2.destroyAllWindows()
