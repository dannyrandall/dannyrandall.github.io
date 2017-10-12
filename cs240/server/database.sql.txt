PRAGMA foreign_keys = 0;
.mode 'column';

/* drop old tables */
drop table if exists persons;
drop table if exists events;
drop table if exists auth;
drop table if exists users;

PRAGMA foreign_keys = 1;

/* create tables */
create table users
(
	id text NOT NULL PRIMARY KEY,
	username text NOT NULL,
	password text NOT NULL,
	email text NOT NULL,
	first_name text NOT NULL,
	last_name text NOT NULL,
	gender text NOT NULL 
		CONSTRAINT ck_gender check (gender in ('m', 'f')),
	person_id text
		REFERENCES persons(id)
		ON UPDATE CASCADE
		ON DELETE SET NULL
);

create table persons
(
	id text NOT NULL PRIMARY KEY,
	descendant_id text NOT NULL 
		REFERENCES users(id)
		ON UPDATE CASCADE
		ON DELETE CASCADE,
	first_name text NOT NULL,
	last_name text NOT NULL,
	gender text NOT NULL
		CONSTRAINT ck_gender check (gender in ('m', 'f')),
	father_id text
		REFERENCES persons(id)
		ON UPDATE CASCADE
		ON DELETE SET NULL,
	mother_id text
		REFERENCES persons(id)
		ON UPDATE CASCADE
		ON DELETE SET NULL,
	spouse_id text
		REFERENCES persons(id)
		ON UPDATE CASCADE
		ON DELETE SET NULL
);

create table events
(
	id text NOT NULL PRIMARY KEY,
	descendant_id text NOT NULL
		REFERENCES users(id)
		ON UPDATE CASCADE
		ON DELETE CASCADE,
	person_id text NOT NULL
		REFERENCES persons(id)
		ON UPDATE CASCADE
		ON DELETE CASCADE,
	latitude integer NOT NULL,
	longitude integer NOT NULL,
	country text NOT NULL,
	city text NOT NULL,
	type text NOT NULL,
	year integer NOT NULL
);

create table auth
(
	id integer NOT NULL PRIMARY KEY autoincrement,
	user_id text NOT NULL
		REFERENCES users(id)
		ON UPDATE CASCADE
		ON DELETE CASCADE,
	token text NOT NULL	
);

/* insert some test data */
insert into users (id, username, password, email, first_name, last_name, gender) values ('user1', 'username', 'password', 'test@sql.com', 'Person', 'lastname', 'm');

insert into persons (id, descendant_id, first_name, last_name, gender) values ('person1', 'user1', 'Dad', 'lastname', 'm');
insert into persons (id, descendant_id, first_name, last_name, gender) values ('person2', 'user1', 'Mom', 'lastname', 'f');
insert into persons (id, descendant_id, first_name, last_name, gender) values ('person3', 'user1', 'Wife', 'lastname', 'f');
insert into persons (id, descendant_id, first_name, last_name, gender, father_id, mother_id, spouse_id) values ('person4', 'user1', 'Person', 'lastname', 'm', 'person1', 'person2', 'person3');

update users 
	set person_id = 'person4'
	where id = 'user1';

update persons
	set spouse_id = 'person4' 
	where id = 'person3';
