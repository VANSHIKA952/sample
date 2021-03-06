import os
from PIL import Image   #PIL is the python image library which needs to be preinstalled before you can run this code
import random
from tkinter import*
from tkinter.filedialog import askopenfilename
chr_ascii={}        #dictionary conatining characters as keys and ascii values as values
ascii_chr={}        #dictionary containing ascii values as keys and characters as values
class Application(Frame):
    def _init_(self,master=none):
        self.filename=" "
        Frame._init_(self,master)
        self.pack()
        self.create_widgets()
    def open_file(self):
        self.file_name = askopenfilename(filetypes=[('BMP File', '*.bmp')])

        # change label text dynamically
        self.name_label['text'] = 'Name: ' + self.file_name

        # clear message in msg_box
        if self.msg_box.get('1.0', END) != '':
            self.msg_box.delete('1.0', END)

        # clear image file
        # img & photo must be global or image will not show
        global left_img
        left_img = None
        global left_photo
        left_photo = None
        global right_img
        right_img = None
        global right_photo
        right_photo = None

        left_img = Image.open(self.file_name)
        w, h = left_img.size
        self.dimensions_label['text'] = 'Dimensions: ' + str(w) + 'x' + str(h)

        self.size_label['text'] = 'Size: ' + str(os.path.getsize(self.file_name)) + 'Bytes'

        # mode https://pillow.readthedocs.io/en/4.1.x/handbook/concepts.html#concept-modes
        if left_img.mode == 'L':
            self.mode_label['text'] = 'Mode: 8-Bit Pixels, Black and White'
            self.available = int((w * h) / 8 - 4)
            if self.available <= 0:
                self.available_label['text'] = 'Available Size For Stega: 0 Bytes'
            else:
                self.available_label['text'] = 'Available Size For Stega: ' + str(self.available) + 'Bytes'
        elif left_img.mode == 'RGB':
            self.mode_label['text'] = 'Mode: 3x8-Bit Pixels, True Color'
            self.available = int((w * h * 3) / 8 - 4)
            if self.available <= 0:
                self.available_label['text'] = 'Available Size For Stega: 0 Bytes'
            else:
                self.available_label['text'] = 'Available Size For Stega: ' + str(self.available) + 'Bytes'
        else:
            self.mode_label['text'] = 'Mode: ' + left_img.mode

        # resize img
        scale_w = img_display_width / w
        scale_h = img_display_height / h
        scale = min(scale_w, scale_h)
        new_w = math.ceil(scale * w)
        new_h = math.ceil(scale * h)
        # Image.NEAREST http://pillow.readthedocs.io/en/4.1.x/releasenotes/2.7.0.html
        left_img = left_img.resize((new_w, new_h), Image.NEAREST)

        left_photo = ImageTk.PhotoImage(left_img)

        self.left_img_canvas.create_image(img_display_width / 2, img_display_height / 2, anchor=CENTER,
                                          image=left_photo)

    for i in range(0,256):
        chr_ascii[chr(i)]=i
        ascii_chr[i]=chr(i)
    
    def encrypt():
        img=input("\nEnter the name of the image you want the message to be encoded in, with the extension\n\n")
        path=os.path.abspath(img)    #this gives the absolute path of the given file
        image=Image.open(path)
        height=image.height
        width=image.width
        password=input("\n\nEnter security key\n\n")
        text=input("\n\nEnter the message you wanna encode\n\n")
        p=0
        j=0
        
        z=0
        text_length=len(text)
        image.putpixel((0,0),(text_length//255, text_length%255,0))
        random.seed(password)
        x = random.sample(range(1,width+1), text_length)
        y = random.sample(range(1, height+1), text_length)
        coor = list(zip(x,y))
        for i in range(text_length):
            RGB=image.getpixel(coor[i])
            rgb_mutable=list(RGB)
            rgb_mutable[z]=chr_ascii[text[p]]^chr_ascii[password[j]]
            rgb_tuple=tuple(rgb_mutable)
            image.putpixel(coor[i],rgb_tuple)
            
            
            z=(z+1)%3
            j=(j+1)%len(password)
            p+=1
            
        image.show()
        image.save(img[:-4]+"_encrypt.png")
        print("\n\nData hiding in image is successful\n\n")
    def decrypt():
        img=input("\nEnter the name of the image you want the message to be encoded in, with the extension\n\n")
        path=os.path.abspath(img)    #this gives the absolute path of the given file
        image=Image.open(path)
        height=image.height
        width=image.width
        j1=0
        
        z1=0
        password=input("\n\nEnter your key\n\n")
        message=''
        raw=list(image.getpixel((0,0)))
        length = raw[0]*255 + raw[1]
        
        random.seed(password)
        x = random.sample(range(1,width+1), length)
        y = random.sample(range(1, height+1), length)
        coor = list(zip(x,y))
        for i in range(length):
            RGB2=image.getpixel(coor[i])
            coded=RGB2[z1]
            
            message+=ascii_chr[coded^chr_ascii[password[j1]]]
    
            z1=(z1+1)%3
            j1=(j1+1)%len(password)
            '''
            RGB2=image.getpixel((x1,y1))
            coded=RGB2[z1]
    
            message+=ascii_chr[coded^chr_ascii[password[j1]]]
                
                
            z1=(z1+1)%3
            j1=(j1+1)%len(password)
            '''
        print("\n\nEncrypted text was\n\n",message)   
    user = input("Type 'encrypt' if you want encryption \nType 'decrypt' if you want decryption\n" )
    if user == 'encrypt':
        encrypt()
    elif user == 'decrypt':
        decrypt()
    else:
        
     def create_widgets(self):
        # do not try to use grid and pack in the same window

        # left part -----------------------------------------------------------
        left_frame = Frame(self)
        left_frame.pack(side=LEFT)

        show_frame = Frame(left_frame)
        show_frame.pack(side=TOP)

        open_frame = Frame(show_frame)
        open_frame.pack(side=TOP)

        open_label = Label(open_frame, text='Open BMP File:')
        open_label.pack(side=LEFT)

        open_button = Button(open_frame, text='Open', command=self.open_file)
        open_button.pack(side=LEFT)

        self.name_label = Label(show_frame, text='Name: ')
        self.name_label.pack(side=TOP)

        self.dimensions_label = Label(show_frame, text='Dimensions: ')
        self.dimensions_label.pack(side=TOP)

        self.size_label = Label(show_frame, text='Size: ')
        self.size_label.pack(side=TOP)

        self.mode_label = Label(show_frame, text='Available Size For Stega: ')
        self.mode_label.pack(side=TOP)

        self.available_label = Label(show_frame, text='Mode: ')
        self.available_label.pack(side=TOP)

        self.left_img_canvas = Canvas(left_frame, bg='grey', width=img_display_width, height=img_display_height)
        self.left_img_canvas.pack(side=BOTTOM)

        # right part ------------------------------------------------------
        right_frame = Frame(self)
        right_frame.pack(side=RIGHT)

        en_de_cry_frame = Frame(right_frame)
        en_de_cry_frame.pack(side=TOP)

        decry_button = Button(en_de_cry_frame, text='Decryption', command=decrypt)
        decry_button.pack(side=LEFT)

        encry_button = Button(en_de_cry_frame, text='Encryption', command=encrpyt)
        encry_button.pack(side=RIGHT)

        msg_frame = Frame(right_frame)
        msg_frame.pack(side=TOP)

        self.msg_box = Text(msg_frame, width=42, height=7)
        self.msg_box.pack(side=BOTTOM)

        # right button part ---------------------------------------------------
        self.right_img_canvas = Canvas(right_frame, bg='grey', width=img_display_width, height=img_display_height)
        self.right_img_canvas.pack(side=BOTTOM)


left_img = None
left_photo = None
right_img = None
right_photo = None
img_display_width = 300
img_display_height = 200
app = Application()
app.master.title('LSB Steganography')
app.mainloop()
