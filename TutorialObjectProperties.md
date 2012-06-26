# Creating a Game Object Properties
<a name="top" />
At the core of every game engine is the inevitable Game Object or Game Entity. Many game developers find that each Game Object/Entity need special properties (e.g. hit points, speed, velocity, bullets, etc.). Some of these properties are unique for a specific Game Object/Entity and others are used for each Game Objects/Entity. The challenge is creating code that can deal with both situations without causing too much complexity in your Game Mechanics (or rules that define game interactions between each Game Object/Entity). For the [[GQE Project|http://code.google.com/p/gqe/]] we have developed a simple but powerful PropertyManager class and IProperty and TProperty holder classes. By using only these three classes we can customize each Game Object/Entity easily without resorting to messy Inheritance or complex compositions. This way the game rules can simply detect if a specific property is available in the interacting entity and tweak its value as necessary.

###Quick Links###

1. [PropertyManager](#manager)
2. [IProperty](#iproperty)
3. [TProperty](#tproperty)
4. [Comments](#comments)

## <a name="manager" />PropertyManager [ [Top] ](#top)
The purpose of the PropertyManager class is to hold all IProperty derived classes and provide the ability to add, get, and set specific properties. By using template meta programming techniques we can make this very easy and flexible for the programmer. A few typical calls to the PropertyManager would look like this:

```cpp
#include "PropertyManager.hpp"

int main(int argc, char* argv[])
{
  PropertyManager properties;

  // Add a new sf::Clock property
  properties.Add<sf::Clock>("AnimationClock", sf::Clock());

  // Add a new sf::Sprite property
  properties.Add<sf::Sprite>("Sprite", sf::Sprite());

  // Add a boolean property
  properties.Add<bool>("Visible", true);

  // Get the Visible property
  if(properties.Get<bool>("Visible") == true)
  {
    // Draw our sprite
    renderWindow.draw(properties.Get<sf::Sprite>("Sprite"));

    // Reset our Animation Clock
    properties.Get<sf::Clock>("AnimationClock").reset();
  }

  // See if a property exists
  if(properties.HasID("NotExist"))
  {
    printf("It exists!\n");
  }
  else
  {
    printf("It doesn't exist\n");
  }

  return 0;
}
```

The beauty of this system is that it is very easy to save nearly any kind of property you might want to store in your Game Object/Entity. Each Game Entity can have unique properties added without creating an inheritance class nightmare. This makes your game logic reusable and more generic. Here is the .hpp file for the PropertyManager class
```cpp
#ifndef PROPERTY_MANAGER_HPP_INCLUDED
#define PROPERTY_MANAGER_HPP_INCLUDED

#include <map>
#include <typeinfo>
#include "IProperty.hpp"
#include "TProperty.hpp"

namespace GQE
{
  /// Provides the PropertyManager class for managing IProperty classes
  class PropertyManager
  {
    public:
      /**
       * PropertyManager default constructor
       */
      PropertyManager();

      /**
       * PropertyManager deconstructor
       */
      virtual ~PropertyManager();

      /**
       * HasProperty returs true if thePropertyID specified exists in this
       * PropertyManager.
       * @param[in] thePropertyID to lookup in this PropertyManager
       * @return true if thePropertyID exists, false otherwise
       */
      bool HasID(const std::string thePropertyID);

      /**
       * GetProperty returns the property as type with the ID of thePropertyID.
       * @param[in] thePropertyID is the ID of the property to return.
       * @return the value stored in the found propery in the form of TYPE. If no
       * Property was found it returns the default value the type constructor.
       */
      template<class TYPE>
      TYPE Get(const std::string thePropertyID)
      {
        if(mList.find(thePropertyID) != mList.end())
        {
          if(mList.at(thePropertyID)->GetType()->Name() == typeid(TYPE).name())
            return static_cast<TProperty<TYPE>*>(mList[thePropertyID])->GetValue();
        }
        else
        {
          WLOG() << "PropertyManager:Get() returning blank property("
            << thePropertyID << ") type" << std::endl;
        }
        TYPE anReturn=TYPE();
        return anReturn;
      }

      /**
       * SetProperty sets the property with the ID of thePropertyID to theValue.
       * @param[in] thePropertyID is the ID of the property to set.
       * @param[in] theValue is the value to set.
       */
      template<class TYPE>
      void Set(const std::string thePropertyID, TYPE theValue)
      {
        if(mList.find(thePropertyID) != mList.end())
        {
          if(mList.at(thePropertyID)->GetType()->Name() == typeid(TYPE).name())
          {
            static_cast<TProperty<TYPE>*>(mList[thePropertyID])->SetValue(theValue);
          }
        }
        else
        {
          ELOG() << "PropertyManager:Set() unable to find property("
            << thePropertyID << ")" << std::endl;
        }
      }

      /**
       * AddProperty Creates a Property and addes it to this entity.
       * @param[in] thePropertyID is the ID of the property to create.
       * @param[in] theValue is the inital value to set.
       */
      template<class TYPE>
      void Add(const std::string thePropertyID, TYPE theValue)
      {
        // Only add the property if it doesn't already exist
        if(mList.find(thePropertyID) == mList.end())
        {
          TProperty<TYPE>* anProperty=new(std::nothrow) TProperty<TYPE>(thePropertyID);
          anProperty->SetValue(theValue);
          mList[anProperty->GetID()]=anProperty;
        }
      }

      /**
       * AddProperty gets a premade Property and addes it to this entity.
       * @param[in] theProperty is a pointer to a pre exisiting property.
       */
      void Add(IProperty* theProperty);

      /**
       * Clone is responsible for making a clone of each property in the
       * PropertyManager provided.
       * @param[in] thePropertyManager to clone into ourselves
       */
      void Clone(const PropertyManager& thePropertyManager);

    protected:

    private:
      // Variables
      ///////////////////////////////////////////////////////////////////////////
      /// A map of all Properties available for this PropertyManager class
      std::map<const std::string, IProperty*> mList;
  }; // PropertyManager class
} // namespace GQE
#endif
```
…and for the .cpp file you have the following:
```cpp
#include "PropertyManager.hpp"

namespace GQE
{
  PropertyManager::PropertyManager()
  {
  }

  PropertyManager::~PropertyManager()
  {
    // Make sure to remove all registered properties on desstruction
    std::map<const std::string, IProperty*>::iterator anProptertyIter;
    for(anProptertyIter = mList.begin();
        anProptertyIter != mList.end();
        ++anProptertyIter)
    {
      IProperty* anProperty = (anProptertyIter->second);
      delete anProperty;
      anProperty = NULL;
    }
  }

  bool PropertyManager::HasID(const std::string thePropertyID)
  {
    bool anResult = false;

    // See if thePropertyID was found in our list of properties
    anResult = (mList.find(thePropertyID) != mList.end());

    // Return true if thePropertyID was found above, false otherwise
    return anResult;
  }


  void PropertyManager::Add(IProperty* theProperty)
  {
    if(mList.find(theProperty->GetID()) == mList.end())
    {
      mList[theProperty->GetID()] = theProperty;
    }
  }

  void PropertyManager::Clone(const PropertyManager& thePropertyManager)
  {
    // Make sure to remove all registered properties on desstruction
    std::map<const std::string, IProperty*>::const_iterator anProptertyIter;
    for(anProptertyIter = thePropertyManager.mList.begin();
        anProptertyIter != thePropertyManager.mList.end();
        ++anProptertyIter)
    {
      IProperty* anProperty = (anProptertyIter->second);
      Add(anProperty->MakeClone());
    }
  }
} // namespace GQE
```
## <a name="iproperty" />IProperty [ [Top] ](#top)

The IProperty interface class is necessary to allow the PropertyManager to hold the same kind of class. Even though the PropertyManager makes use of templates, we still need this base class to exist to make it easier for the PropertyManager to hold a std::map of each property. Here is the .hpp file for the IProperty interface class:
```cpp
#ifndef IPROPERTY_HPP_INCLUDED
#define IPROPERTY_HPP_INCLUDED

namespace GQE
{
  /// Provides the interface for all IEntity properties
  class IProperty
  {
    public:
      /// Class that represents the type for this class
      class Type_t
      {
        private:
          std::string mName;
        public:
          explicit Type_t(std::string theName) : mName(theName) {}
          std::string Name() const
          {
            return mName;
          };
      };

      /**
       * IProperty default constructor
       * @param[in] theType of property this property represents
       * @param[in] thePropertyID to use for this property
       */
      IProperty(std::string theType, const std::string thePropertyID);

      /**
       * IProperty destructor
       */
      virtual ~IProperty();

      /**
       * GetType will return the Type_t type for this property
       * @return the Type_t class for this property
       */
      Type_t* GetType(void);

      /**
       * GetID will return the Property ID used for this property.
       * @return the property ID for this property
       */
      const std::string GetID(void) const;

      /**
       * Update will be called for each IProperty registered with IEntity and
       * enable each IProperty derived class to perform Update related tasks
       * (e.g. frame counter, timer update, decreate in shields, etc).
       */
      virtual void Update() = 0;

      /**
       * MakeClone is responsible for creating a clone of this IProperty
       * derived class and returning it as part of the Prototype and Instance
       * system. The value of the Property will also be copied into the clone.
       * @return pointer to the IProperty derived class clone that was created
       */
      virtual IProperty* MakeClone() = 0;

    protected:
      /**
       * SetType is responsible for setting the type of class this IProperty
       * class represents and is usually called by the IProperty derived class
       * to set theType.
       * @param[in] theType to set for this IProperty derived class
       */
      void SetType(std::string theType);

    private:
      // Variables
      ///////////////////////////////////////////////////////////////////////////
      /// The type that represents this class
      Type_t mType;
      /// The property ID assigned to this IProperty derived class
      const std::string mPropertyID;
  };
} // namespace GQE
#endif //IPROPERTY_HPP_INCLUDED
```
…and for the .cpp file you have the following:
```cpp
#include "IProperty.hpp"

namespace GQE
{
  IProperty::IProperty(std::string theType, const std::string thePropertyID) :
    mType(theType),
    mPropertyID(thePropertyID)
  {
  }

  IProperty::~IProperty()
  {
  }

  IProperty::Type_t* IProperty::GetType(void)
  {
    return &mType;
  }

  const std::string IProperty::GetID(void) const
  {
    return mPropertyID;
  }

  void IProperty::SetType(std::string theType)
  {
    mType = Type_t(theType);
  }
} // namespace GQE
```

## <a name="tproperty" />TProperty [ [Top] ](#top)
And now for the TProperty template class that makes the magic happen. Template classes are funny in that you must provide the implementation in the .hpp file (this is the most compatible way of creating Template classes across multiple compilers). Because this class inherits from IProperty we can store each custom property in a single std::map variable in the PropertyManager class shown above. And by using a little bit of functional template programming in the PropertyManager class we can provide a standard interface for retrieving custom properties.
```cpp
#ifndef TPROPERTY_HPP_INCLUDED
#define TPROPERTY_HPP_INCLUDED

#include <typeinfo>
#include "IProperty.hpp"

namespace GQE
{
  /// The Template version of the IProperty class for custom property values
  template<class TYPE=unsigned int>
    class TProperty : public IProperty
  {
    public:
      /**
       * TProperty default constructor
       * @param[in] thePropertyID to use for this property
       */
      TProperty(const std::string thePropertyID) :
        IProperty(typeid(TYPE).name(), thePropertyID)
    {
    }

      /**
       * GetValue will return the property value
       * @return the property value
       */
      TYPE GetValue()
      {
        return mValue;
      }

      /**
       * SetValue will set the property value to the value
       * provided.
       */
      void SetValue(TYPE theValue)
      {
        mValue=theValue;
      }

      /**
       * Update will be called giving each property a chance to change itself
       * during the Update phase of the GameLoop (see IEntity::UpdateFixed)
       */
      void Update()
      {
      }

      /**
       * MakeClone is responsible for creating a clone of this IProperty
       * derived class and returning it as part of the Prototype and Instance
       * system. The value of the Property will also be copied into the clone.
       * @return pointer to the IProperty derived class clone that was created
       */
      IProperty* MakeClone()
      {
        TProperty<TYPE>* anProperty = new(std::nothrow) TProperty<TYPE>(GetID());

        // Make sure new didn't fail before setting the value for this property
        if(NULL != anProperty)
        {
          anProperty->SetValue(mValue);
        }

        // Return cloned anProperty or NULL if none was created
        return anProperty;
      }
    private:
      TYPE mValue;
  };
} // namespace GQE
#endif // TPROPERTY_HPP_INCLUDED
```
Well this concludes the PropertyManager introduction and hopefully you will find the technique as useful in your game projects as we have for ours.

## <a name="comments" />Comments [ [Top] ](#top)

//Ryan Lindeman//