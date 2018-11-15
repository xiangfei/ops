    




  下载LOFTER我的照片书  |
public class Order {
    @id
    private long id;
 
    @ManyToOne
    @JoinColumn(name="customer", nullable=false)
    private Customer customer;
 
    public void setId(long id) {
        this.id = id;
    }
 
    public long getId() {
        return this.id;
    }
 
    public void setCustomer(Customer customer) {
        this.customer = customer;
    }
 
    public Customer getCustomer() {
        return this.customer;
    }
}
 
public class Customer{
    @id
    private long id;
 
    @OneToMany(mappedBy="customer")
    private Set<Order> orders;
 
    public void setId(long id) {
        this.id = id;
    }
 
    public long getId() {
        return this.id;
    }
 
    public void setOrders(Set<Order> orders) {
        this.orders = orders;
    }
 
    public Set<Order> getOrders() {
        return this.orders;
    }
}
if we don't use joincolumn and manppedby  jpa will generated a relative table