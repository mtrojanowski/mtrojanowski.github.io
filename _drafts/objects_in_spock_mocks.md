In Spock be careful when using mocks.

myEntity = new Entity()

then:
1 * repositoryMock.saveEntity(myEntity) >> myEntity

Can not work if myEntity is not the same object as the one passed to saveEntity (check why!). 
Use `saveEntity(_)` instead!


