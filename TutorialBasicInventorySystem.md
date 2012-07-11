### **Scheduled for editing to put more information into it!**

Hey, here's a basic inventory system that (as a beginner) I'm fairly proud of. Just a little disclaimer, if you're a good programmer then this post won't be of use to you, as you'd be able to come up with a totally better system including pointers, iterations etc etc. *This is programmed in SFML 1.6 but can _easily_ be updated to SFML2! I also know that things can be done a lot better than how I've gone about them.*

This post is more aimed at the newer people to SFML, whom have gone over the first tutorials but are seeking out examples of such.

Down to business.

For my inventory system, I had a total of 6 functions: InventoryDisplay, InventoryAdd, InventoryRemove, InventoryUpdate, InventoryEquip and InventoryUnequip as well as a few global variables.

The first thing to do is to create some items to work with; this is using the normal [sprite creation tutorial](http://sfml-dev.org/tutorials/1.6/graphics-sprite.php).

### **--- Global Variables ---**

I'll now shed some light on the global variables:

```
int Slot[48] = {0};

int Item1Have[3] = {0}; int Item3Have[3] = {0};
int Item2Have[3] = {0}; int Item4Have[3] = {0};

int Gold = 0;

int SwordEquipped = 0; int OffHandEquipped = 0;
int BracersEquipped = 0; int HandsEquipped = 0;
int HelmetEquipped = 0; int ChestEquipped = 0; int LegsEquipped = 0;
```

Here we have an array for the inventory slots (48 of them), some more arrays for the different items then integers for Gold, then equipped armours and weapons.

Slot[48]: I use this array to tell the program which slots have been taken. We'll focus more on this in InventoryAdd.

Item1Have[3]: This array is for Item1 and has 3 values of 0. Item1Have[0] is to tell the program if that item is in your inventory. Item1Have[1] stores the slot that it is in. Item1Have[2] tells the program if it has been equipped.

The rest are pretty self explanatory.

### **--- InventoryDisplay ---**

This is the function to call when you press the inventory key:
```
if( (Input.GetMouseX() >= 545 && Input.GetMouseX() <= 567 && Input.GetMouseY() >= 165 && Input.GetMouseY() <= 197) || (Input.GetMouseX() >= 545 && Input.GetMouseX() <= 567 && Input.GetMouseY() >= 205 && Input.GetMouseY() <= 237) ||
            (Input.GetMouseX() >= 545 && Input.GetMouseX() <= 567 && Input.GetMouseY() >= 85 && Input.GetMouseY() <= 117) || (Input.GetMouseX() >= 545 && Input.GetMouseX() <= 567 && Input.GetMouseY() >= 125 && Input.GetMouseY() <= 157) ||

           (Input.GetMouseX() >= 675 && Input.GetMouseX() <= 707 && Input.GetMouseY() >= 85 && Input.GetMouseY() <= 117) || (Input.GetMouseX() >= 675 && Input.GetMouseX() <= 707 && Input.GetMouseY() >= 145 && Input.GetMouseY() <= 177) ||
           (Input.GetMouseX() >= 675 && Input.GetMouseX() <= 707 && Input.GetMouseY() >= 205 && Input.GetMouseY() <= 237)||

           (Input.GetMouseX() >= 780 && Input.GetMouseX() <= 807 && Input.GetMouseY() >= 85 && Input.GetMouseY() <= 117) || (Input.GetMouseX() >= 780 && Input.GetMouseX() <= 807 && Input.GetMouseY() >= 125 && Input.GetMouseY() <= 157) ||
           (Input.GetMouseX() >= 780 && Input.GetMouseX() <= 807 && Input.GetMouseY() >= 165 && Input.GetMouseY() <= 197) || (Input.GetMouseX() >= 780 && Input.GetMouseX() <= 807 && Input.GetMouseY() >= 205 && Input.GetMouseY() <= 237) ||

           (Input.GetMouseX() >= 55  && Input.GetMouseX() <= 87  && (Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242 || Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282 || Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322 || Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362 || Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402 || Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442)) ||
           (Input.GetMouseX() >= 95  && Input.GetMouseX() <= 127 && (Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242 || Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282 || Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322 || Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362 || Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402 || Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442)) ||
           (Input.GetMouseX() >= 135 && Input.GetMouseX() <= 167 && (Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242 || Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282 || Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322 || Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362 || Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402 || Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442)) ||
           (Input.GetMouseX() >= 175 && Input.GetMouseX() <= 207 && (Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242 || Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282 || Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322 || Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362 || Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402 || Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442)) ||
           (Input.GetMouseX() >= 215 && Input.GetMouseX() <= 247 && (Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242 || Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282 || Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322 || Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362 || Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402 || Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442)) ||
           (Input.GetMouseX() >= 255 && Input.GetMouseX() <= 287 && (Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242 || Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282 || Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322 || Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362 || Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402 || Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442)) ||
           (Input.GetMouseX() >= 295 && Input.GetMouseX() <= 327 && (Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242 || Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282 || Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322 || Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362 || Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402 || Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442)) ||
           (Input.GetMouseX() >= 335 && Input.GetMouseX() <= 367 && (Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242 || Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282 || Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322 || Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362 || Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402 || Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442)))
        {
            int i; int e;
            if(Input.GetMouseX() >= 55 && Input.GetMouseX() <= 87 && Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242) i = 0;
            if(Input.GetMouseX() >= 95 && Input.GetMouseX() <= 127 && Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242) i = 1;
            if(Input.GetMouseX() >= 135 && Input.GetMouseX() <= 167 && Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242) i = 2;
            if(Input.GetMouseX() >= 175 && Input.GetMouseX() <= 207 && Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242) i = 3;
            if(Input.GetMouseX() >= 215 && Input.GetMouseX() <= 247 && Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242) i = 4;
            if(Input.GetMouseX() >= 255 && Input.GetMouseX() <= 287 && Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242) i = 5;
            if(Input.GetMouseX() >= 295 && Input.GetMouseX() <= 327 && Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242) i = 6;
            if(Input.GetMouseX() >= 335 && Input.GetMouseX() <= 367 && Input.GetMouseY() >= 210 && Input.GetMouseY() <= 242) i = 7;

            if(Input.GetMouseX() >= 55 && Input.GetMouseX() <= 87 && Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282) i = 8;
            if(Input.GetMouseX() >= 95 && Input.GetMouseX() <= 127 && Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282) i = 9;
            if(Input.GetMouseX() >= 135 && Input.GetMouseX() <= 167 && Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282) i = 10;
            if(Input.GetMouseX() >= 175 && Input.GetMouseX() <= 207 && Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282) i = 11;
            if(Input.GetMouseX() >= 215 && Input.GetMouseX() <= 247 && Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282) i = 12;
            if(Input.GetMouseX() >= 255 && Input.GetMouseX() <= 287 && Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282) i = 13;
            if(Input.GetMouseX() >= 295 && Input.GetMouseX() <= 327 && Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282) i = 14;
            if(Input.GetMouseX() >= 335 && Input.GetMouseX() <= 367 && Input.GetMouseY() >= 250 && Input.GetMouseY() <= 282) i = 15;

            if(Input.GetMouseX() >= 55 && Input.GetMouseX() <= 87 && Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322) i = 16;
            if(Input.GetMouseX() >= 95 && Input.GetMouseX() <= 127 && Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322) i = 17;
            if(Input.GetMouseX() >= 135 && Input.GetMouseX() <= 167 && Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322) i = 18;
            if(Input.GetMouseX() >= 175 && Input.GetMouseX() <= 207 && Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322) i = 19;
            if(Input.GetMouseX() >= 215 && Input.GetMouseX() <= 247 && Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322) i = 20;
            if(Input.GetMouseX() >= 255 && Input.GetMouseX() <= 287 && Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322) i = 21;
            if(Input.GetMouseX() >= 295 && Input.GetMouseX() <= 327 && Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322) i = 22;
            if(Input.GetMouseX() >= 335 && Input.GetMouseX() <= 367 && Input.GetMouseY() >= 290 && Input.GetMouseY() <= 322) i = 23;

            if(Input.GetMouseX() >= 55 && Input.GetMouseX() <= 87 && Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362) i = 24;
            if(Input.GetMouseX() >= 95 && Input.GetMouseX() <= 127 && Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362) i = 25;
            if(Input.GetMouseX() >= 135 && Input.GetMouseX() <= 167 && Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362) i = 26;
            if(Input.GetMouseX() >= 175 && Input.GetMouseX() <= 207 && Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362) i = 27;
            if(Input.GetMouseX() >= 215 && Input.GetMouseX() <= 247 && Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362) i = 28;
            if(Input.GetMouseX() >= 255 && Input.GetMouseX() <= 287 && Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362) i = 29;
            if(Input.GetMouseX() >= 295 && Input.GetMouseX() <= 327 && Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362) i = 30;
            if(Input.GetMouseX() >= 335 && Input.GetMouseX() <= 367 && Input.GetMouseY() >= 330 && Input.GetMouseY() <= 362) i = 31;

            if(Input.GetMouseX() >= 55 && Input.GetMouseX() <= 87 && Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402) i = 32;
            if(Input.GetMouseX() >= 95 && Input.GetMouseX() <= 127 && Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402) i = 33;
            if(Input.GetMouseX() >= 135 && Input.GetMouseX() <= 167 && Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402) i = 34;
            if(Input.GetMouseX() >= 175 && Input.GetMouseX() <= 207 && Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402) i = 35;
            if(Input.GetMouseX() >= 215 && Input.GetMouseX() <= 247 && Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402) i = 36;
            if(Input.GetMouseX() >= 255 && Input.GetMouseX() <= 287 && Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402) i = 37;
            if(Input.GetMouseX() >= 295 && Input.GetMouseX() <= 327 && Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402) i = 38;
            if(Input.GetMouseX() >= 335 && Input.GetMouseX() <= 367 && Input.GetMouseY() >= 370 && Input.GetMouseY() <= 402) i = 39;

            if(Input.GetMouseX() >= 55 && Input.GetMouseX() <= 87 && Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442) i = 40;
            if(Input.GetMouseX() >= 95 && Input.GetMouseX() <= 127 && Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442) i = 41;
            if(Input.GetMouseX() >= 135 && Input.GetMouseX() <= 167 && Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442) i = 42;
            if(Input.GetMouseX() >= 175 && Input.GetMouseX() <= 207 && Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442) i = 43;
            if(Input.GetMouseX() >= 215 && Input.GetMouseX() <= 247 && Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442) i = 44;
            if(Input.GetMouseX() >= 255 && Input.GetMouseX() <= 287 && Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442) i = 45;
            if(Input.GetMouseX() >= 295 && Input.GetMouseX() <= 327 && Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442) i = 46;
            if(Input.GetMouseX() >= 335 && Input.GetMouseX() <= 367 && Input.GetMouseY() >= 410 && Input.GetMouseY() <= 442) i = 47;

           if(Input.GetMouseX() >= 545 && Input.GetMouseX() <= 567 && Input.GetMouseY() >= 85 && Input.GetMouseY() <= 117) e = 0; // Bracers
           if(Input.GetMouseX() >= 545 && Input.GetMouseX() <= 567 && Input.GetMouseY() >= 125 && Input.GetMouseY() <= 157) e = 1; // Hands
           if(Input.GetMouseX() >= 545 && Input.GetMouseX() <= 567 && Input.GetMouseY() >= 165 && Input.GetMouseY() <= 197) e = 2; // Sword
           if(Input.GetMouseX() >= 545 && Input.GetMouseX() <= 567 && Input.GetMouseY() >= 205 && Input.GetMouseY() <= 237) e = 3; // Off hand

           if(Input.GetMouseX() >= 675 && Input.GetMouseX() <= 707 && Input.GetMouseY() >= 85 && Input.GetMouseY() <= 117) e = 4; // Helmet
           if(Input.GetMouseX() >= 675 && Input.GetMouseX() <= 707 && Input.GetMouseY() >= 145 && Input.GetMouseY() <= 177) e = 5; // Chest
           if(Input.GetMouseX() >= 675 && Input.GetMouseX() <= 707 && Input.GetMouseY() >= 205 && Input.GetMouseY() <= 237)  e = 6; // Legs

           if(Input.GetMouseX() >= 780 && Input.GetMouseX() <= 807 && Input.GetMouseY() >= 85 && Input.GetMouseY() <= 117) e = 7; // Neck 1
           if(Input.GetMouseX() >= 780 && Input.GetMouseX() <= 807 && Input.GetMouseY() >= 125 && Input.GetMouseY() <= 157) e = 8; // Neck 2
           if(Input.GetMouseX() >= 780 && Input.GetMouseX() <= 807 && Input.GetMouseY() >= 165 && Input.GetMouseY() <= 197) e = 9; // Ring 1
           if(Input.GetMouseX() >= 780 && Input.GetMouseX() <= 807 && Input.GetMouseY() >= 205 && Input.GetMouseY() <= 237) e = 10; // Ring 2

            // Item Equip/Use
            if(Item1Have[0] == 1 && Item1Have[1] == i && SwordEquipped != 1 && ((Event.Type == sf::Event::MouseButtonPressed) && (Event.MouseButton.Button == sf::Mouse::Left))) { InventoryEquip(1); if(Quest3 == 1 && Quest4 == 0 && g_QuestStatus == 1){g_QuestStatus = 2; } }

            // Armour Unequip
            if(e == 2 && SwordEquipped == 1 && Item1Have[2] == 1 &&((Event.Type == sf::Event::MouseButtonPressed) && (Event.MouseButton.Button == sf::Mouse::Left))) { InventoryUnequip(1); }

            // Equipment delete
            if(App.GetInput().IsKeyDown(sf::Key::D) && ((Event.Type == sf::Event::MouseButtonPressed) && (Event.MouseButton.Button == sf::Mouse::Right)))
            {
                if(Item1Have[0] == 1 && Item1Have[1] == i)  InventoryRemove(1);
                if(Item2Have[0] == 1 && Item2Have[1] == i)  InventoryRemove(2);
                if(Item3Have[0] == 1 && Item3Have[1] == i)  InventoryRemove(3);
                if(Item4Have[0] == 1 && Item4Have[1] == i)  InventoryRemove(4);
            }

            // Item Descriptions
            if((Item1Have[0] == 1 && Item1Have[1] == i) || SwordEquipped == 1 && e == 2) {Description.SetText("A simple shortsword.\n+1 Physical AP.\n\n(Using this item will equip/unequip it)"); scrollOver = 1;}
            if(Item2Have[0] == 1 && Item2Have[1] == i) {Description.SetText("A health potion."); scrollOver = 1;}
            if(Item3Have[0] == 1 && Item3Have[1] == i) {Description.SetText("A mana potion."); scrollOver = 1;}
            if(Item4Have[0] == 1 && Item4Have[1] == i) {Description.SetText("Pet Rock.\nYour pet rock will always love you."); scrollOver = 1;}
        }
        else { Description.SetText(""); scrollOver = 0; }
```

**NOTE:** It's to note that inventory spaces are 32x32 and the massive if statement is check whether the mouse is inside any of these spaces, then it changes the variable 'i' to reflect which space is being hovered over or 'e' for an equipped piece of armour.

I'm not going to go through everything here (unless someone is confused), but look mainly at the way I equip, unequip or delete armour.

### **--- InventoryAdd ---**

Here is the way to add an item to the inventory. To actually add something to the inventory during the game, just call InventoryAdd(2) the number being the item.

```
int InventoryAdd(int ItemID)
{
            int i = 0;
            int x = 0;
            int y = 0;
            while(i < 49)
            {
                if(Slot[i] == 0)
                {
                if(i == 0) {x = 55; y = 210;}
                if(i == 1) {x = 95; y = 210;}
                if(i == 2) {x = 135; y = 210;}
                if(i == 3) {x = 175; y = 210;}
                if(i == 4) {x = 215; y = 210;}
                if(i == 5) {x = 255; y = 210;}
                if(i == 6) {x = 295; y = 210;}
                if(i == 7) {x = 335; y = 210;}

                if(i == 8) {x = 55; y = 250;}
                if(i == 9) {x = 95; y = 250;}
                if(i == 10) {x = 135; y = 250;}
                if(i == 11) {x = 175; y = 250;}
                if(i == 12) {x = 215; y = 250;}
                if(i == 13) {x = 255; y = 250;}
                if(i == 14) {x = 295; y = 250;}
                if(i == 15) {x = 335; y = 250;}

                if(i == 16) {x = 55; y = 290;}
                if(i == 17) {x = 95; y = 290;}
                if(i == 18) {x = 135; y = 290;}
                if(i == 19) {x = 175; y = 290;}
                if(i == 20) {x = 215; y = 290;}
                if(i == 21) {x = 255; y = 290;}
                if(i == 22) {x = 295; y = 290;}
                if(i == 23) {x = 335; y = 290;}

                if(i == 24) {x = 55; y = 330;}
                if(i == 25) {x = 95; y = 330;}
                if(i == 26) {x = 135; y = 330;}
                if(i == 27) {x = 175; y = 330;}
                if(i == 28) {x = 215; y = 330;}
                if(i == 29) {x = 255; y = 330;}
                if(i == 30) {x = 295; y = 330;}
                if(i == 31) {x = 335; y = 330;}

                if(i == 32) {x = 55; y = 370;}
                if(i == 33) {x = 95; y = 370;}
                if(i == 34) {x = 135; y = 370;}
                if(i == 35) {x = 175; y = 370;}
                if(i == 36) {x = 215; y = 370;}
                if(i == 37) {x = 255; y = 370;}
                if(i == 38) {x = 295; y = 370;}
                if(i == 39) {x = 335; y = 370;}

                if(i == 40) {x = 55; y = 410;}
                if(i == 41) {x = 95; y = 410;}
                if(i == 42) {x = 135; y = 410;}
                if(i == 43) {x = 175; y = 410;}
                if(i == 44) {x = 215; y = 410;}
                if(i == 45) {x = 255; y = 410;}
                if(i == 46) {x = 295; y = 410;}
                if(i == 47) {x = 335; y = 410;}

                if(ItemID == 1) { Item1.SetPosition(x,y); Item1Have[0] = 1; Item1Have[1] = i; Slot[i] = 1; return 0; }
                if(ItemID == 2) { Item2.SetPosition(x,y); Item2Have[0] = 1; Item2Have[1] = i; Slot[i] = 1; return 0; }
                if(ItemID == 3) { Item3.SetPosition(x,y); Item3Have[0] = 1; Item3Have[1] = i; Slot[i] = 1; return 0; }
                if(ItemID == 4) { Item4.SetPosition(x,y); Item4Have[0] = 1; Item4Have[1] = i; Slot[i] = 1; return 0; }
                }
                i++;
            }
}
```

The way this works, is it declares some variables, then it going into the while loop (while (i < 49)) then it goes through each slot and asks if it's available, if it isn't, then 'i' is incremented and then the next slot is tested for availability. 

If a slot is available (lets say i = 2 so slot 3 is available, then it puts x and y to where slot 3 is, then it gets the ItemID, sets the position of the item, tells it that it's been added, puts in the slot it's been added to, fills the slot then ends the function with 'return 0'.

### **--- InventoryRemove ---**
```
int InventoryRemove(int ItemID)
{
    if(ItemID == 1) {Item1Have[0] = 0; Slot[Item1Have[1]] = 0; Item1Have[1] = 0; return 0;}
    if(ItemID == 2) {Item2Have[0] = 0; Slot[Item2Have[1]] = 0; Item2Have[1] = 0; return 0;}
    if(ItemID == 3) {Item3Have[0] = 0; Slot[Item3Have[1]] = 0; Item3Have[1] = 0; return 0;}
    if(ItemID == 4) {Item4Have[0] = 0; Slot[Item4Have[1]] = 0; Item4Have[1] = 0; return 0;}
}
```
This then removes an item from the program supplying an ItemID when the function is called. It resets everything to tell the program that that slot is now free for another item to fill it's place.

### **--- InventoryUpdate ---**
```
int InventoryUpdate()
{
    Health.SetText(convertInt(Stats[0]));
    Mana.SetText(convertInt(Stats[1]));
    Stamina.SetText(convertInt(Stats[2]));
    Defense.SetText(convertInt(Stats[3]));
    MagicResist.SetText(convertInt(Stats[4]));
    PhysicalAP.SetText(convertInt(Stats[5]));
    MagicalAP.SetText(convertInt(Stats[6]));
}
```
This is what updates the stats of the character when an item is equipped, as they are shown in the inventory screen. This makes sure that when an item is equipped, that it will show the new stats straight away.

### **--- InventoryEquip & InventoryUnequip ---**
```
int InventoryEquip(int ItemID)
{
    if(ItemID == 1) { SwordEquipped = 1; InventoryRemove(1); Item1Have[2] = 1; Stats[5] += 1; InventoryUpdate(); }
}
```

This will equip the certain item, in this case, a sword. It changes the variable SwordEquipped to 1, to tell that that slot is taken then it removes it from the inventory, changes the Item2Have[2] variable to show it's been equipped then gives the stat change and updates it!

Whereas, the InventoryUnequip does the exact opposite:
```
int InventoryUnequip(int ItemID)
{
    if(ItemID == 1) { SwordEquipped = 0; InventoryAdd(1); Item1Have[2] = 0; Stats[5] -= 1; InventoryUpdate(); }
}
```

Hopefully this can help some of the beginners of SFML (and coding) to give them an idea of one way to implement a basic inventory. Remember to change it to your project's needs!
