---
layout: post
title: "Refactoring Legacy Code - Reducing coupling"
tags: 
- Java
- Refactory
- English posts
---
{% include JB/setup %}

I'm working in a Java Legacy Code and facing high coupling and low cohesion, kind of trivial problems in this scenario.

I needed to refactor the code, not only to improve the design, but in this case was firstly to make it unit testable. The design improvements started as a "side effect".

This post concerns about the baby steps that I used to refactor the code.

## Legacy code

    public class OrderDAO {
    	public boolean delete(int id) {
    	// ...
    	}
    }
    
    public class OrderService {
    	public boolean delete(int id) {
    		return new OrderDAO().delete(id);
    	}
    }
    
    public class OrderServiceTest {
    	@Test
    	public void shouldDeleteAnOrder() {
    		OrderService subject = new OrderService();
    		assertTrue(subject.delete(1));
    	}
    }

* OrderService is high coupled with OrderDAO

* OrderDAO will be created all times that delete is called

It is impossible to unit test it. I can’t mock the OrderDAO because it is initiated within the method.

The test is an integration test, not unit test. To run it, I need to provide a datasource... and I don’t want to.

## First refactory

    public class OrderService {
    	public boolean delete(int id) {
    		return getOrderDAO().delete(id);
    	}
    	protected OrderDAO getOrderDAO() {
    		return new OrderDAO();
    	}
    }
    
    public class OrderServiceTest {
    	@Test
    	public void shouldDeleteAnOrder() {
    		final OrderDAO orderDAO = mock(OrderDAO.class);
    		doReturn(true).when(orderDAO).delete(anyInt());
    		/*
    		 * Anonymous class rocks
    		*/
    		OrderService subject = new OrderService(){
    			protected OrderDAO getOrderDAO(){
    				return orderDAO;
    			}
    		};
    		assertTrue(subject.delete(1));
    	}
    }

Great! It isn’t the state of the art, but at least I can mock the OrderDAO and there isn’t a side effect. I don’t need to change the classes that use OrderService. Its behaviour is still the same.

Mockito has a convenient method called <a href="http://mockito.googlecode.com/svn/branches/1.6/javadoc/org/mockito/Mockito.html#spy(T)">spy</a>, that can be used instead of the Anonymous class, but I don't feel confortable with the idea of using a test subject from Mockito/CGLIB.

    OrderDAO orderDAO = mock(OrderDAO.class);
    doReturn(true).when(orderDAO).delete(anyInt());
    OrderService subject = spy(OrderService.class);
    doReturn(orderDAO).when(subject).getOrderDAO();
    assertTrue(subject.delete(1));

## Second refactory

    public class OrderService {
    	private OrderDAO orderDAO;
    
    	public OrderService() {
    		orderDAO = new OrderDAO();
    	}
    	public OrderService(OrderDAO orderDAO) {
    		this.orderDAO = orderDAO;
    	}
    	public boolean delete(int id) {
    		return orderDAO.delete(id);
    	}
    }
    
    public class OrderServiceTest {
    	@Test
    	public void shouldDeleteAnOrder() {
    		final OrderDAO orderDAO = mock(OrderDAO.class);
    		doReturn(true).when(orderDAO).delete(anyInt());
    		OrderService subject = new OrderService(orderDAO);
    		assertTrue(subject.delete(1));
    	}
    }

A little bit better, still having a strong dependency of OrderDAO, but at least it will be created once.

## Third refactory

    public interface Persistable {
    	public boolean delete(int id);
    }
    
    public class OrderDAO implements Persistable {
    	public boolean delete(int id) {
            // ...
    	}
    }
    
    public class OrderService {
    	private Persistable orderDAO;
    
    	public OrderService() {
    		orderDAO = new OrderDAO();
    	}
    	public OrderService(Persistable orderDAO) {
    		this.orderDAO = orderDAO;
    	}
    	public boolean delete(int id) {
    		return orderDAO.delete(id);
    	}
    }
    
    public class OrderServiceTest {
    	@Test
    	public void shouldDeleteAnOrder() {
    		final Persistable orderDAO = mock(Persistable.class);
    		doReturn(true).when(orderDAO).delete(1);
    		OrderService subject = new OrderService(orderDAO);
    		assertTrue(subject.delete(1));
    	}
    }

The strong dependency of OrderDAO is optional. It's just for backward compatibility.


## Next refactory?

I stopped here, not because I think that it is over, but I don’t want to be too pragramatic. I improved it a little and the most important is that I mantained the backward compatibility.

My probable next step could be to add [Contexts and Dependency Injection](http://jcp.org/en/jsr/detail?id=299) CDI. The motivation to use the Dependency Injection is to encapsulate the object creation, as a class that uses an object doesn’t need to know how to create it. Other point is that DI frameworks have contexts, such as Request, Session and Application, and it reduces unnecessary creations of objects.

I wrote a gist with these steps [gist.github.com/3108659](https://gist.github.com/3108659).
