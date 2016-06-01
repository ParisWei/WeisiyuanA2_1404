# WeisiyuanA2_1404

ItemListGUI

BoxLayout:
    #Parent layout
    id: mainBox
    Popup:
        id: popup
        size_hint: (.7, .7)
        title: "Add Phonebook Entry"
        on_parent:
            # Popup wont be visible when the program starts
            if self.parent == mainBox: self.parent.remove_widget(self)
        GridLayout:
            # GUI of the popup
            id: inputs
            cols:2
            rows:4
            orientation: 'vertical'
            Label:
                text: 'Item name: '
                size_hint_y: .25

            TextInput:
                id: newName
                value: ''
                size_hint_y: None
                multiline: False #These lines will make the cursor enter the next textbox when tab is pressed
                write_tab: False

            Label:
                text: 'Description: '
                size_hint_y: .25

            TextInput:
                id: newDesc
                value: ''
                size_hint_y: None

                multiline: False
                write_tab: False

            Label:
                text: 'Cost per day: '
                size_hint_y: .25

            TextInput:
                id: newCost
                value: ''
                size_hint_y: None
                multiline: False
                write_tab: False
            Button:
                text: 'Save Entry'
                on_release: app.create_new_item(newName.text, newDesc.text, newCost.text)
            Button:
                text: 'Cancel'
                on_release: app.cancel_new_item()
            Label:
                text: app.messageBar
    BoxLayout:
    #Main menu layout
        orientation: 'vertical'
        BoxLayout:
            orientation: 'horizontal'
            BoxLayout:
                orientation: 'vertical'
                size_hint_x: 0.3

                Button:
                    id: listMenu
                    text: 'List item'
                    on_release: app.click_menuList(self)

                Button:
                    id: hireMenu
                    text: 'Hire item'
                    on_release: app.click_menuHire(self)

                Button:
                    id: returnMenu
                    text: 'Return item'
                    on_release: app.click_menuReturn(self)

                Button:
                    id: confirmMenu
                    text: 'Confirm'
                    on_release: app.confirmItems()

                Button:
                    id: addItem
                    text: 'Add item'
                    on_release: app.add_new_item()

                Button:
                    id: quitMenu
                    text: 'Quit'
                    on_release: app.click_Quit()

            GridLayout:
                # this layout is where the item buttons are created
                id: entriesBox
                cols: 2
                orientation: 'vertical'
        Label:
            id: messageBar #This is where item information and errors are displayed
            size_hint_y: 0.1
            text: app.messageBar

A1.py

from kivy.app import App
from kivy.lang import Builder
from kivy.uix.button import Button
from kivy.properties import StringProperty
import csv

class itemsHireApp(App):
    def __init__(self, **kwargs):
        self.saveList = self.operateCsv('r',[])
        self.totalCost = 0.0
        self.changeList = []
        self.itemList = {}
        self.menuChoice = '1'
        super(itemsHireApp, self).__init__(**kwargs)
        
        for i in range(len(self.saveList)):
            self.itemList[self.saveList[i][0]] = self.saveList[i][1:]
        
    messageBar = StringProperty()
        
    def build(self):
        self.title = "Items for Hire"
        self.root = Builder.load_file('ItemListGUI.kv')
        self.create_item_buttons()
        self.root.ids.listMenu.state = 'down'
        return self.root
        
    def operateCsv(self, rw, savelist):
        with open('./items.csv', rw, newline='') as csvfile:
            if rw == 'r':
                reader = csv.reader(csvfile, delimiter=',')
                dataList = [item for item in reader]
                return dataList
            if rw == 'w':
                writer = csv.writer(csvfile, delimiter=',')
                writer.writerows(savelist)
                return
     
    def create_item_buttons(self):
        for key in self.itemList:
            newButton = Button(text = key)
            newButton.bind(on_release = self.button_click)
            if self.itemList[key][2] == 'out':
                newButton.background_color = (1, .5, 1, 0.5)
            else:
                newButton.background_color = (.5, 1, .5, 1)
            self.root.ids.entriesBox.add_widget(newButton)
            
    def button_click(self, instance):
        if self.menuChoice == 'l':
            #self.messageBar = "{}: {}. ${} per day.  Item is currently {}".format(instance.text, self.itemList[instance.text][0], self.itemList[instance.text][1], self.itemList[instance.text][2]
            #self.messageBar = "nothing to do"
            self.messageBar = "{}:{}. ${} per day. Item is {}".format(instance.text, self.itemList[instance.text][0], self.itemList[instance.text][1], self.itemList[instance.text][2])
        elif self.menuChoice == 'h':
            #nothing to do
           #self.messageBar = 'choos item to hire'
            self.items_to_change(instance, 'in')
        elif self.menuChoice == 'r':
            self.messageBar = 'Which item you want to return'
            self.items_to_change(instance, 'out')
            
    def click_menuList(self, instance):
        self.messageBar = 'Select an item to add list'
        self.click_Menu(instance.text)
        instance.state = 'down'
        self.menuChoice = 'l'
        
    def click_Menu(self, menu):
        if menu!= 'Hire item':
            self.root.ids.hireMenu.state = 'normal'
        if menu!= 'Return item':
            self.root.ids.returnMenu.state = 'normal'
        if menu!= 'List item':
            self.root.ids.listMenu.state = 'normal'
            
    def click_menuHire(self, instance):
        self.messageBar = 'Select an item to hire'
        self.click_Menu(instance.text)
        instance.state='down'
        self.menuChoice = 'h'
        
        
    def click_menuReturn(self, instance):
        self.messageBar = 'Select an item to return'
        self.click_Menu(instance.text)
        instance.state = 'down'
        self.menuChoice = 'r'
        
    def click_menuList(self, instance):
        self.messageBar = 'Select an item to list'
        self.click_Menu(instance.text)
        instance.state = 'down'
        self.menuChoice = 'l'
        
    def items_to_change(self, name, inout):
        if self.itemList[name.text][2] == inout:
            name.state = 'down'
            if name not in self.changeList:
                self.changeList.append(name)
                self.totalCost += float(self.itemList[name.text][1])
            elif name.state == 'down':
                self.totalCost -= float(self.itemList[name.text][1])
                name.state = 'normal'
                self.changeList.remove(name)
        else:
            self.messageBar = 'This item is not available now'
        if self.menuChoice == 'h':
            self.messageBar = "This will cost ${:.2f} per day".format(self.totalCost)
    
    def confirmItems(self):
        for name in self.changeList:
            self.confirmChanges(name)
        self.changeList = []
     
    def confirmChanges(self, name):
        name.state = 'normal'
        if self.menuChoice == 'h':
            self.itemList[name.text][2] = 'out'
            name.background_color = (1, .5, 1, .5)
            for i in range(len(self.saveList)):
                if self.saveList[i][0] == name.text:
                    self.saveList[i][3] = 'out'
        elif self.menuChoice == 'r':
            self.itemList[name.text][2] = 'in'
            name.background_color = (.5, 1, .5, 1)
            for i in range(len(self.saveList)):
                if self.saveList[i][0] == name.text:
                    self.saveList[i][3] = 'in'
                    
    def add_new_item(self):
        self.messageBar = "Enter new item's data"
        self.root.ids.popup.open()
        
    def cancel_new_item(self):
        self.messageBar = 'nothing to do'
        
    def cancel_new_item(self):
        self.root.ids.popup.dismiss()
        self.clearAll()
        self.messageBar = ""
    
    def clearAll(self):
        self.root.ids.newValue = []
        self.root.ids.newName.text = ''
        self.root.ids.newDesc.text = ''
        self.root.ids.newCost.text = ''
                    
                    
    def click_Quit(self):
        self.operateCsv('w', self.saveList)
        itemsHireApp.stop(self)
        
    def create_new_item(self, newName, newDesc, newCost):
        if len(newName)>0 and len(newDesc)>0 and float(newCost)>-1:
            newValue = []
            newValue.append(newName)
            newValue.append(newDesc)
            newValue.append(newCost)
            newValue.append('in')
            self.saveList.append(newValue)
            self.itemList[newName] = newValue[1:]
            
            newButton = Button(text=newName)
            newButton.bind(on_release=self.button_click)
            newButton.background_color = (.5, 1, .5, 1)
            self.root.ids.entriesBox.add_widget(newButton)
            self.root.ids.popup.dismiss()
            self.clearAll()
   

if __name__=='__main__':
    itemsHireApp().run()
