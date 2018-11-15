# -*- coding: utf-8 -*-
'''
Created on 2015年8月31日

@author: fei.xiang
'''

import threading
import os

from Queue import Queue

class ScanFolderThread(threading.Thread):
    '''
    classdocs
    '''

    
    def __init__(self,dir_name,file_queue,file_type_list): 
        threading.Thread.__init__(self) 
        self.daemon = True 
        self.dir_name = dir_name 
        self.file_type_list = file_type_list
        self.file_queue = file_queue
        
    def run(self): 

        for root,dirs,files in os.walk(self.dir_name):
            for filespath in files:
                for file_type in self.file_type_list.split("#"):
                    file_name = os.path.join(root,filespath)
                    #print file_name
                    if file_name.find(file_type)!= -1 and file_name.find(".svn") == -1:
                        #print os.path.join(root,filespath)
                        self.file_queue.put(os.path.join(root,filespath))
                        break
                    
        
class ScanAndWriteContentThread(threading.Thread):
    '''
    classdocs
    '''


    def __init__(self,input_file_queue , file_name,contain_type,file_content, thread_lock): 
        threading.Thread.__init__(self) 
        self.daemon = True 
        self.input_file_queue = input_file_queue 
        self.file_name = file_name 
        self.contain_type = contain_type
        self.file_content = file_content
        self.thread_lock = thread_lock
        
    def run(self): 
        while True: 

            if self.input_file_queue.empty(): 
                break 
            
            input_file = self.input_file_queue.get()
            content = ""
            if self.thread_lock.acquire():
                with open(input_file , 'r') as fw:
                    content = fw.read()
                if self.contain_type == 0:
                    if content == "" or  content.find(self.file_content) != -1 :
                        with open(self.file_name ,'a+') as fw:
                            fw.write(input_file + "\n") 
                        
                elif self.contain_type == 1:
                    
                    if content == "" or  content.find(self.file_content) == -1:
                        with open(self.file_name ,'a+') as fw:
                            fw.write(input_file + "\n") 
                else :
                    raise Exception("unknown content type")
                self.thread_lock.release()
                self.input_file_queue.task_done()





def scan_folder(base_dir_name, file_type_list, contain_type, file_content , new_file_name):
    
    dir_list = [ f for f in os.listdir(base_dir_name) if os.path.isdir(os.path.join(base_dir_name,f)) ]
    
    print dir_list
    lock = threading.Semaphore()
#remove if file exists
    if os.path.exists(new_file_name):
        os.remove(new_file_name)
        
    input_file_queue = Queue()

    threads = []

    for d in dir_list:
        if d.find(".svn") != -1 :
            continue
        threads.append(ScanFolderThread(os.path.join(base_dir_name,d),input_file_queue,file_type_list))
        
    for x in range(4):
        threads.append(ScanAndWriteContentThread(input_file_queue,new_file_name,contain_type ,file_content,lock))

    for thread in threads:
        thread.start()

    for thread in threads:
        thread.join()

    
if __name__ == '__main__':
    scan_folder("E:\\app-ctmp", "xml#java", 1 , "test" ,  "E:\\scan_result/python.log")

缺点，不加锁，文件格式错误， 加锁，性能变慢
改进，可以写入不同的文件，在写一个多线程处理。