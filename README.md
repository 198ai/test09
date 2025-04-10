@startuml
class B {
  - displayType
  + setDisplayType(displayType)
  + adjust〇〇Layout(margin)
}
class Fragment1 extends B {
  + onViewCreated()
  + adjust〇〇Layout(margin)
}
class Fragment2 extends B {
  + onViewCreated()
  + adjust〇〇Layout(margin)
}

enum DisplayType {
  Type1
  Type2
}


note right of Fragment2::onViewCreated
  呼び出す setDisplayType(DisplayType.Type2)
end note
note right of Fragment1::onViewCreated
  呼び出す setDisplayType(DisplayType.Type1)
end note
@enduml
