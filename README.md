#Merchily.tn
First affiliation patform in Tunisia 🇹🇳 http://www.merchily.tn
Made with:
- Python Django Back-end
- HTML/Bootsrap Front-end

### Features

- Four types of Users (Super-Admin / Staff-Admin / Vendor / Business)
- Vendor and Business users:
	- Has available Balance
	- Had pending Balance
	- Can Edit Profile infos
	- User Can be Postpaid Or Prepaid
- Business:
	- Add Products
	- Manage Stock
	- Recive Orders
	- Downloads Invoice/ delivery form
	- Check Balance
- Vendor:
	- Discover Products
	- Select product by adding it to favorite List to start selling them
	- Add Orders by selecting Products from Favorite List
	- Check Balance
- Orders automticly added to delivery system by API
- Delivery Form Generated from delivery system
- Order Status is auto-updated
	- Non Confirmed
	- Pending
	- In Process
	- In Delivery
	- Livred
	- Return
	- Payed
- Each new user is affected to staff memeber when signup (can be changed once a day by user)
- Staff-Admin:
	- Can confirm new users affected to them
	- Can confirm new product of affected user
	- Can manage affected users
- Super-Admin:
	- Staff-Admin roles but with all users
	- Can make deposit and withdraws
- Dark Mode

####Code Sppinet

Vendor Discover Product Can make search and pagination


    def SearchProductView(request):
		"""Discover product"""
		if request.user.is_authenticated:
			if request.user.is_vendor:
				query = ""
				if request.GET.get("q", ""):
					query = request.GET["q"]
				if query != "":
					prod = search_product(query=query)
				else:
					prod_one = Products.objects.filter(is_confirmed=True)
					prod = prod_one.filter(publishing_status=True)
				try:
					user_fav = Favorite_list.objects.get(vendor=request.user)
				except Favorite_list.DoesNotExist:
					user_fav = Favorite_list.objects.create(vendor=request.user)
				user_prod = user_fav.product.all()

				my_prod_list = sorted(prod, key=attrgetter('id'), reverse=True)

				page = request.GET.get('page', 1)
				paginator = Paginator(my_prod_list, 10)
				try:
					products = paginator.page(page)
				except PageNotAnInteger:
					products = paginator.page(1)
				except EmptyPage:
					products = paginator.page(paginator.num_pages)

				return render(request, 'vendor/search-product-list.html', {
					'products': products,
					'query': str(query),
					'user_prod': user_prod
				})
    


####Code Sppinet 2 

Vendor Add Product to fav list by making a POST request to API

js jquery

        $(document).on('click', '.fav-prod', function() {
			let Products = []
			let ProductId = $(this).attr('id').split("-").pop();
			Products.push(ProductId);
			$.ajax({
				type: 'POST',
				headers: {"X-CSRFToken": csrftoken},
				dataType: "json",
				contentType: "application/json",
				data: JSON.stringify({'product': Products}),
				url: '/api/new-fav/',
				success: function ( response ) {
					if (response.response === "added") {
						$('#fav-prod-' + ProductId).empty();
						$('#fav-prod-' + ProductId).append('<em class="icon icon-circle bg-primary-dim icon ni ni-star-fill"></em>');
						toastr.clear();
						NioApp.Toast('Ce produit a ete ajouté au favoris', 'success');

					} else {
						$('#fav-prod-' + ProductId).empty();
						$('#fav-prod-' + ProductId).append('<em class="icon icon-circle bg-primary-dim icon ni ni-star"></em>');
						toastr.clear();
						NioApp.Toast('Ce produit a ete supprimé du favoris', 'warning');
					}
				}
			});
    });
    


Django RestFrame Work 

        def post(self, request, *args, **kwargs):
			"""check if an object already exist"""
			my_product = Products.objects.get(id=request.data["product"][0])
			try:
				fav = Favorite_list.objects.get(vendor_id=request.user.id)
				if Products.objects.get(id=request.data["product"][0]) in fav.product.all():
					fav.product.remove(request.data["product"][0])
					fav.save()
					my_product.current_vendor_number -= 1
					my_product.save()
					return JsonResponse({'response': 'removed'})
				else:
					fav.product.add(request.data["product"][0])
					fav.save()
					my_product.current_vendor_number += 1
					my_product.save()
					return JsonResponse({'response': 'added'})
			except:
				my_product.current_vendor_number -= 1
				my_product.save()
				return self.create(request, *args, **kwargs)
    



# Screen Shots

![](https://i.imgur.com/z4jqwy2.png)
![](https://i.imgur.com/e2cuemX.png)

Business Add Product:
![](https://i.imgur.com/73Nk8TT.png)


Vendor see product:
![](https://i.imgur.com/MyzoU4D.png)

![](https://i.imgur.com/HKQBPum.png)

![](https://i.imgur.com/b5CktiC.png)

![](https://i.imgur.com/508vX0O.png)

![](https://i.imgur.com/Kznz5SW.png)

Admin:
![](https://i.imgur.com/6RvNRuX.png)
![](https://i.imgur.com/Ux6u7qk.png)


