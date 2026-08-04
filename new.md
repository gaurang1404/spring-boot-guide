fetching strategies

eager loading when immediately needed, default for
onetoone and manytoone

lazy loading when related objects are loaded when accessed

oneToMany
ManyToMany

fetch = FetchType.LAZY, but should be added to the entity that is the owner

For
one to one, if user has profile, profile is owner of relationship and that is where this should be added
for one to one, if we dont need profile while fetching user, remove the
private Profile profile altogether

If fetching profile while lazy loading is enabled, and later if you want to getUser, make the method @Transactional, since the persistence context is needed to fetch the items that are lazy loaded

If we create a user object and an address object and setAddress for the user and userRepository.save(user), the address will not get saved automatically

you will have to use an address repository to do so but that is very unclean

we have cascade types that we can set to persist related objects

explain other important types with examples