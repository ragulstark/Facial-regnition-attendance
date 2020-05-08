#facial-recognition


import cv2, sys, numpy, os 
from playsound import playsound
size = 4
haar_file = "haarcascade_frontalface_alt2.xml"
datasets = 'datasets'
  
# Part 1: Create FineRecognizer 
print('Recognizing Face Please Be in sufficient Lights...') 
  
# Create a list of images and a list of corresponding names 
(images, lables, names, id) = ([], [], {}, 0) 
for (subdirs, dirs, files) in os.walk(datasets): 
    for subdir in dirs: 
        names[id] = subdir 
        subjectpath = os.path.join(datasets, subdir) 
        for filename in os.listdir(subjectpath): 
            path = subjectpath + '/' + filename 
            lable = id
            images.append(cv2.imread(path, 0)) 
            lables.append(int(lable)) 
        id += 1
(width, height) = (130, 100) 
  
# Create a Numpy array from the two lists above 
(images, lables) = [numpy.array(lis) for lis in [images, lables]] 
  
# OpenCV trains a model from the images 
# NOTE FOR OpenCV2: remove '.face' 
model = cv2.face.LBPHFaceRecognizer_create() 
model.train(images, lables) 
  
# Part 2: Use fisherRecognizer on camera stream 
face_cascade = cv2.CascadeClassifier(haar_file) 
webcam = cv2.VideoCapture(0) 
sname=""
re=1
li=[]
while True: 
    (_, im) = webcam.read() 
    gray = cv2.cvtColor(im, cv2.COLOR_BGR2GRAY) 
    faces = face_cascade.detectMultiScale(gray, 1.3, 5) 
    for (x, y, w, h) in faces: 
        cv2.rectangle(im, (x, y), (x + w, y + h), (255, 0, 0), 2) 
        face = gray[y:y + h, x:x + w] 
        face_resize = cv2.resize(face, (width, height)) 
        # Try to recognize the face 
        prediction = model.predict(face_resize) 
        cv2.rectangle(im, (x, y), (x + w, y + h), (0, 255, 0), 3) 
  
        if prediction[1]<100:
            cv2.putText(im, '% s - %.0f' % (names[prediction[0]], prediction[1]), (x-10, y-10),  cv2.FONT_HERSHEY_PLAIN, 1, (0, 255, 0))
            re+=1
            sname=names[prediction[0]]
            if re==30:
                import sqlite3
                conn = sqlite3.connect('regis.db')
                r=conn.cursor()
                r.execute('select * from regis where name=?',(sname,))
                rows=r.fetchall()
                for i in rows:
                    li.append(i)
                print(li)
                table_name = 'atta'
                conn = sqlite3.connect('regis.db')
                c = conn.cursor()
                # sql = 
                c.execute('create table if not exists ' + table_name + ' (name varchar(50),rollno varchar(50),class varchar(50),attanance varchar(50))')
                c.execute('insert into '+table_name+'  values (?,?,?,"present")',(li[0][0],li[0][1],li[0][2]))
                conn.commit()
                conn.close()
                print("attanance marked")
                playsound('preview.mp3')
            
        else:
            cv2.putText(im, 'not recognized',(x-10, y-10), cv2.FONT_HERSHEY_PLAIN, 1, (0, 255, 0))
            
    cv2.imshow('OpenCV', im) 
      
    key = cv2.waitKey(10) 
    if key == 27:
        break
    
    
webcam.release()
cv2.destroyAllWindows()
