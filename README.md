# Algorithm - Date

## table of contents
1. [Show tomorrow date](#show-tomorrow-date)
2. [Define Date.nextDay using prototype](#)


## Show tomorrow date
```js
var today = new Date();
var tomorrow = new Date();
tomorrow.setDate(today.getDate()+1);

console.log(today);//Thu Dec 08 2016 08:35:42 GMT-0800 (PST)
console.log(tomorrow); //Fri Dec 09 2016 08:32:48 GMT-0800 (PST)
```

//cart_ui.js
```js
(function () {
    'use strict';
    var isShoppingCart = function () {
        return $('#ajx_shopping_cart').length;
    }();
    var isSecureCheckout = function () {
        return $('#ajx_checkout_cart').length;
    }();
    /*
      Checkout Page source code.
    */
    var initSecureCheckout = function () {
        var cartElements = {
            checkoutWrapper: $('#one_step_checkout'),
            checkoutForm: $('#payment_form'),
            inputs: $('#one_step_checkout').find('.form-ctrl'),
            paymentSelector: $('#payment_methods').find('.pm-select'),
            newCustomerChk: $('#is_new_customer'),
            copyToShippingChk: $('#is_same_shipping'),
            processOrderBtn: $('.place-order-btn'),
            billingForm: $('#checkout_billing'),
            shippingForm: $('#checkout_shipping'),
            shipMethodsList: $('#shipping_methods'),
            shippingSelector: $('#shipping_methods').find('.rdo-ship'),
            shipState: $('.ajx-ship-state'),
            shipZipCode: $('.ajx-ship-zip'),
            pdEligibleChk: $('#eligible_for_pd'),
            pdApplyBtn: $('.ajx-apply-pd'),
            cc: $('#credit_card'), //cc payment wrapper
            ccInputs: $('#credit_card .form-ctrl'),
            ccExpMonth: $('#ccm'), //cc month expy
            ccExpYear: $('#ccy'), //cc year expy
            billZipCode: $('.ajx-trigger-blr'),
            billState: $('.ajx-trigger-chg'),
            quickRegisterPass: $('#quick_register'),
            paypalForm: $('#submit_to_paypal'),
            paypalBtn: $('.paypal-btn'),
            poKey: $('.ajx-po-key'),
            poApplyBtn: $('.ajx-apply-po'),
            promoToggle: $('#toggle_promo'),
            promoDiv: $('#promo'),
            promoField: $('.ajx-promocode'),
            gcField: $('.ajx-gc'),
            //applyPromoBtn: $('.ajx-get-promo'),
            //applyGCButton: $('.ajx-apply-gc'),
            cartSummary: $('#cart_summary'),
            instoreList: $('#store_list')
        };
        var cart = {
            /* === CONFIG === */
            config: {
                counter: 0,
                filepath: '/orders_launch/one_step_service.php',
                errorMessage: 'We are sorry, but we cannot process your request. Please contact customer service if you keep seeing this error.'
            },
            /* === HELPER CART FUNCTIONS === */
            errorHandler: {
                throwError: function (errorMessage) {
                    var alertBox = $('#checkout_error');
                    if (alertBox.length) {
                        alertBox.html(errorMessage + '<span class="msg-close">&times</span>');
                        alertBox.message({
                            removeOnClose: true
                        });
                        alertBox.scroll({
                            speed: 200
                        });
                    } else {
                        alertBox = $('<div></div>');
                        alertBox.attr('id', 'checkout_error');
                        alertBox.addClass('msg error-msg');
                        alertBox.html(errorMessage + '<span class="msg-close">&times</span>');
                        alertBox.prependTo('#one_step_checkout');
                        alertBox.message({
                            removeOnClose: true
                        });
                        alertBox.scroll({
                            speed: 200
                        });
                    }
                },
                throwShippingError: function () {
                    $('.js-shipping-error').removeClass('hidden').message();
                    $('.js-shipping-error').scroll({speed: 200});
                },
                hideShippingError: function () {
                    $('.js-shipping-error').addClass('hidden');
                }
            },
            utils: {
                promoErrorTimeout: null,
                lock: function (counter) {
                    // lock/unlock cart processing based on ajax request completion
                    if (counter > 0) {
                        cartElements.processOrderBtn.addClass('disabled');
                    } else if (counter === 0) {
                        cartElements.processOrderBtn.removeClass('disabled');
                    }
                },
                getGrandTotal: function() {
                    var grandTotal = $('#ajx_grand_total').text();
                    return parseFloat(grandTotal);
                },
                setCreditCardExpiry: function () {
                    var currDate = new Date(),
                        currMonth = currDate.getMonth() + 1,
                        currYear = currDate.getFullYear();
                    cartElements.ccExpMonth.val(currMonth);
                    cartElements.ccExpYear.val(currYear);
                },
                clearPassword: function () {
                    // Clear password field if 'is new customer' unchecked.
                    if (!cartElements.newCustomerChk.is(':checked')) {
                        cartElements.quickRegisterPass.val('');
                    }
                },
                trimInput: function () {
                    var input = $(this),
                        trm;
                    // credit card number -- remove all white space.
                    if (input.attr('name') === 'credit_card_number' || input.attr('name') === 'phone') {
                        if (input.attr('name') === 'phone') {
                            trm = input.val().replace(/[\D]+/g, '');
                        } else {
                            trm = input.val().replace(/[^A-Za-z0-9]+/g, '');
                        }
                        trm = $.trim(trm);
                    } else {
                        trm = $.trim(input.val());
                    }
                    input.val(trm);
                },
                initShipInfo: function (billingFields) { // on page load copy over billing to shipping (if 'same as..' checked)
                    billingFields.each(function () {
                        var _self = $(this);
                        cart.utils.populateShippingInfo.call(_self);
                    });
                },
                populateShippingInfo: function () {
                    var billField = $(this),
                        shipField = $(this).data('assoc');
                    $('#' + shipField).val(billField.val());
                },
                copyToShipping: function () {
                    var sameShippingInfo = cartElements.copyToShippingChk.is(':checked'),
                        billingFields = cartElements.billingForm.find('.form-ctrl'),
                        tempVal = '';
                    console.log("HIROKO");
                    console.log(sameShippingInfo);

                    if (sameShippingInfo) {
                        tempVal = cartElements.billZipCode.val();
                        // If same as billing is checked, we populate billing info to shipping info
                        cart.utils.initShipInfo(billingFields);
                        cartElements.shippingForm.addClass('hidden');

                        // Data will copied over to shipping in runtime while editing billing info
                        billingFields.on('change blur', cart.utils.populateShippingInfo);
                        cart.ajx.initShippingMethods(cartElements.billState.val());
                        // Re-initialize shipping prices and exceptions
                        //cart.ajx.initShippingMethods(cartElements.billState.val());
                        cartElements.billState.on('change', function () {
                          var userSelectedState = $(this).val();
                          cart.ajx.initShippingMethods(userSelectedState);
                        });
                        cartElements.billZipCode.on('focus', function () {
                          tempVal = $(this).val();
                        });
                        cartElements.billZipCode.on('blur', function () {
                          if ($(this).val() !== tempVal) {
                            cart.ajx.initEvents();
                          }
                        });
                    } else {
                        tempVal = cartElements.shipZipCode.val();
                        // Stop watching for changes in billing info
                        billingFields.off('change blur');
                        cartElements.shippingForm.removeClass('hidden');

                        cart.ajx.initShippingMethods(cartElements.shipState.val());
                        //cart.ajx.initShippingMethods(cartElements.shipState.val());
                        // Ajax is triggered if shipping info is updated
                        cartElements.shipZipCode.on('focus', function () {
                          tempVal = $(this).val();
                        });
                        cartElements.shipZipCode.on('blur', function () {
                          if ($(this).val() !== tempVal) {
                            cart.ajx.initEvents();
                          }
                        });
                        cartElements.shipState.on('change.exception', function () {
                          var userSelectedState = $(this).val();
                          cart.ajx.initShippingMethods(userSelectedState);
                        });
                    }
                },
                updateCartBalance: function (ajaxResponse) {
                    // On successful ajax response, update cart balance.
                    var pricing = {};
                    var payrollData = ajaxResponse.payroll_deduction_data;
                    if (ajaxResponse.error) {
                        printError('here has been an error calculating checkout pricing.');
                        $('#ajx_product_total').addClass('hidden').text('');
                        $('#ajx_embroidery_charges').addClass('hidden').text('');
                        $('#ajx_discount').addClass('hidden').text('');
                        $('#ajx_shipping').addClass('hidden').text('');
                        $('#ajx_tax').addClass('hidden').text('');
                        $('#ajx_grand_total').addClass('hidden').text('');
                    } else {
                        pricing.productTotal = (Number(ajaxResponse.product_total)).toFixed(2, 10);
                        pricing.embroideryCharges = (Number(ajaxResponse.embroidery_charges)).toFixed(2, 10);
                        pricing.shipping = (Number(ajaxResponse.shipping)).toFixed(2, 10);
                        pricing.tax = (Number(ajaxResponse.tax)).toFixed(2, 10);
                        pricing.discount = (Number(ajaxResponse.discount)).toFixed(2, 10);
                        pricing.balanceDue = (Number(ajaxResponse.balance_due)).toFixed(2, 10);

                        $('#ajx_product_total').removeClass('hidden').text(pricing.productTotal);
                        $('#ajx_embroidery_charges').removeClass('hidden').text(pricing.embroideryCharges);
                        $('#ajx_shipping').removeClass('hidden').text(pricing.shipping);
                        $('#ajx_tax').removeClass('hidden').text(pricing.tax);
                        $('.ajx-promo-discount').text(pricing.discount);
                        // grand total
                        $('#ajx_grand_total').removeClass('hidden').text(pricing.balanceDue);
                        // update payroll balance:

                        try {
                          if (payrollData && payrollData.constructor !== Array) {
                            cart.utils.updatePayrollBalance(ajaxResponse);
                          } else if(payrollData.constructor === Array && payrollData.length) {
                            cart.utils.updatePayrollBalance(ajaxResponse);
                          }
                        } catch(e) {
                          printError('Unable to process payroll deduction info', e);
                        }


                        // 1. check if checkout balance IS ZERO:
                        if (ajaxResponse.the_bill_is_paid) { // 1.2 if balance is zero:
                            // Disable credit card payment
                            cart.utils.creditCardOff();
                        } else {  // 1.3 if balance is NOT zero:
                            // Enable credit card payment
                            cart.utils.creditCardOn();
                        }
                    }
                },
                updateCartPayments: function (ajaxResponse) {
                    var payments = ajaxResponse.payments.data;
                    // Check if payments wrapper exists. If no, create one
                    if (!$('#payments').length) {
                        $('<div></div>')
                            .addClass('uppercase payments')
                            .attr('id', 'payments')
                            .insertAfter('.cart-tax');
                    }
                    // check if there is any payment data from ajax response
                    if (payments.length) {
                        var p,
                            pHTML = '';
                        // Iterate through each payment and render in HTML
                        for (var i = 0, x = payments.length; i < x; i += 1) {
                            p = payments[i];
                            p.amount = Number(p.amount).toFixed(2, 10);
                            //  -- add method id to payment row
                            pHTML += '<div class="payment-item row" data-payment-id="' + p.id + '" data-method-id="' + p.method_id + '">';
                            pHTML += '<span class="ajx-payment-info pull-left"><strong>' + p.descr;
                            if (!$.isEmptyObject(p.payment_name)) {
                                pHTML += ' (' + p.payment_name + ')';
                            }
                            pHTML += ' <button class="ajx-del-payment btn btn--sm btn--gamma" type="button" title="Cancel this payment">&times;</button>';
                            pHTML += ' </strong></span>';
                            pHTML += '<span class="ajx-payment-amount currency discount pull-right">' + p.amount + '</span>';
                            pHTML += '</div>';
                        }
                        $('#payments').html(pHTML);
                    }

                    cart.utils.updateCartBalance(ajaxResponse);
                },
                updateShippingPrices: function (ajaxResponse) {
                    var selectedShipping = $('.rdo-ship:checked'),
                        shippingCode = selectedShipping.val(),
                        x = ajaxResponse.shipping;
                    if (x !== undefined) {
                        selectedShipping.closest('.shipping-method-item').attr('data-ship-price', x[shippingCode]);
                    } else {
                        selectedShipping.closest('.shipping-method-item').attr('data-ship-price', '');
                    }
                },
                creditCardOn: function () {
                    // cartElements.cc.removeClass('hidden')
                    //     .prev('.pm-select').find('.chk-pm').attr('checked', true);
                    $('#cc_form').removeClass('hidden');
                    $('#paid_in_full').addClass('hidden');
                },
                creditCardOff: function () {
                    // cartElements.cc.addClass('hidden')
                    //     .prev('.pm-select').find('.chk-pm').attr('checked', false);
                    $('#cc_form').addClass('hidden');
                    $('#paid_in_full').removeClass('hidden');
                },
                creditCardValidate: function (x) {
                    if (x) {
                        cartElements.ccInputs.removeClass('novalidate');
                    } else {
                        cartElements.ccInputs.addClass('novalidate');
                    }
                },
                updateGCMessage: function (error, message) {
                    if (error) {
                        if (error.class === 'warning') {
                            toastr.warning(message);
                        }
                        else if (error.class === 'error') {
                            toastr.error(message);
                        }
                    } else {
                        toastr.success(message);
                    }
                },
                updatePOMessage: function (error, message) {
                    if (error) {
                        if (error.class === 'warning') {
                            toastr.warning(message);
                        }
                        else if (error.class === 'error') {
                            toastr.error(message);
                        }
                    } else {
                        toastr.success(message);
                    }
                },
                GCPOPassword: {
                    showFor: function (pt) {
                        switch (pt) {
                            case 'gc':
                                $('#gc_password_area').removeClass('hidden');
                                break;
                            case 'po':
                                $('#po_password_area').removeClass('hidden');
                                break;
                            default:
                                printError('[GC_PO_PASSWORD]: Unknown payment type');
                                break;
                        };
                    },
                    hideFor: function (pt) {
                        switch (pt) {
                            case 'gc':
                                $('#gc_password_area').addClass('hidden');
                                $('#gc_password').val('');
                                break;
                            case 'po':
                                $('#po_password_area').addClass('hidden');
                                $('#po_password').val('');
                                break;
                            default:
                                printError('[GC_PO_PASSWORD]: Unknown payment type');
                                break;
                        };
                    },
                },
                confirmPayrollElibility: function () { // make sure employee confirms her eligibility before applying
                    if (cartElements.pdEligibleChk.is(':checked')) {
                        $('.ajx-apply-pd').removeClass('disabled');
                    } else {
                        $('.ajx-apply-pd').addClass('disabled');
                    }
                },
                showPayrollMessage: function (x) {
                    if (x.warning) {
                        toastr.warning('Remaining balance has been applied to your order. Please use another payment method for the remaining balance.');
                    } else {
                        toastr.success('Payroll deduction payment has been applied.');
                    }
                },
                updatePayrollBalance: function (ajaxResponse) {
                    var remainingBalance = parseFloat(ajaxResponse.payroll_deduction_data.balance_remaining).toFixed(2);
                    if (typeof remainingBalance !== 'undefined') {
                        $('#pd_balance').text(remainingBalance);
                        if(ajaxResponse.the_bill_is_paid) {
                            $('#pd_show_pay_buttons').addClass('hidden');
                            cart.utils.showPayrollMessage({warning: false});
                        } else if (remainingBalance == 0.00 && !ajaxResponse.the_bill_is_paid) {
                            $('#pd_show_pay_buttons').addClass('hidden');
                            cart.utils.showPayrollMessage({warning: true});
                        } else {
                            $('#pd_show_pay_buttons').removeClass('hidden');
                        }
                    }
                },
                getAllShippingMethods: function () {
                    var smArray = [];
                    cartElements.shipMethodsList
                        .find('.rdo-ship')
                        .each(function () {
                            smArray.push($(this).val());
                        });
                    return smArray.join(',');
                },
                togglePaymentMethods: function () {
                    var pm = $(this);
                    var chk = pm.find('.chk-pm');
                    var isChecked = chk.is(':checked');
                    var collapsePayment = chk.data('collapse-payment');
                    if (isChecked) {
                        // Credit card is a default payment method
                        // and should never be hidden
                        if (collapsePayment !== 'credit_card') {
                            chk.prop('checked', false);
                            $('#' + collapsePayment).addClass('hidden');
                        }
                    } else {
                        chk.prop('checked', true);
                        $('#' + collapsePayment).removeClass('hidden');
                    }
                },
                populatePaypalInfo: function (response) {
                    var paypalData = response.paypal_info;
                    $('#pp_first_name').val(paypalData.first_name);
                    $('#pp_last_name').val(paypalData.last_name);
                    $('#pp_address1').val(paypalData.address1);
                    $('#pp_address2').val(paypalData.address2);
                    $('#pp_state').val(paypalData.state);
                    $('#pp_zip').val(paypalData.zip);
                    $('#pp_city').val(paypalData.city);
                    $('#pp_email').val(paypalData.email); // paypal customer email
                    $('#pp_phone').val(paypalData.phone); // paypal phone number
                    $('#pp_business').val(paypalData.paypal_email); // paypal business email
                    $('#pp_amount_1').val(paypalData.amount_1); // cart amount
                    $('#pp_cpp_header_image').val(paypalData.cpp_header_image);
                    $('#pp_custom').val(paypalData.custom);
                    $('#pp_invoice').val(paypalData.invoice); // invoice #
                    $('#pp_item_name_1').val(paypalData.item_name_1);
                    $('#pp_shopping_url').val(paypalData.shopping_url);
                    $('#pp_use_paypal').val(paypalData.use_paypal);
                    $('#pp_cancel_return').val(response.paypal_info.cancel_return);
                    if (cartElements.paypalForm.length) {
                        if (paypalData.pp_test_action) {
                            cartElements.paypalForm.attr('action', paypalData.pp_test_action);
                        }
                        cartElements.paypalForm.submit();
                    }
                }
            }, // end of utils

            /* === AJAX CALLS ===*/
            ajx: {
                initShippingMethods: function (currSelShipMethod) {
                    // initialize shipping methods on page load. This function will only be called once.
                    // Function takes one optional parameter (shipping state)
                    var shipMethodsList = cartElements.shipMethodsList,
                        counter = cart.config.counter,
                        shipStateSelected = currSelShipMethod || cartElements.shipState.val(),
                        shipMethod, shipMethodSelect, shipCode, shipName, shipPrice, exceptionObj;

                    counter += 1;

                    $.ajax({
                        url: cart.config.filepath,
                        data: {
                            what_to_do: 'shipping_method_cost',
                            shipping_method: cart.utils.getAllShippingMethods,
                            ship_state: cartElements.shipState.val(),
                            zip_code: cartElements.shipZipCode.val()
                        },
                        success: function (response) {
                            if (typeof response === 'object') {
                                shipMethodsList.find('.shipping-method-item').each(function () {
                                    shipMethod = $(this);
                                    shipMethodSelect = shipMethod.find('.rdo-ship');
                                    shipCode = shipMethodSelect.val();
                                    shipName = shipMethod.attr('data-ship-name');
                                    shipPrice = response.shipping[shipCode];
                                    // 1. Initialize prices for ALL shipping methods
                                    if (shipPrice !== null || typeof shipPrice !== 'undefined') {
                                        if (shipPrice > 0) {
                                            shipMethod.attr('data-ship-price', '$' + shipPrice);
                                        } else {
                                            shipMethod.attr('data-ship-price', 'FREE');
                                        }
                                    } else {
                                        shipMethod.attr('data-ship-price', '');
                                    }
                                });
                                try {
                                    shipMethodsList.find('.shipping-method-item').each(function () {
                                        var method = $(this),
                                            exceptionstates = [],
                                            shipName = $(this).data('shipName'),
                                            shipCode = $(this).data('shipCode');
                                        if (response.shipping_exceptions) {
                                            exceptionObj = response.shipping_exceptions;
                                            if (exceptionObj[shipName]) {
                                                if (exceptionObj[shipName][shipCode]) {
                                                    method.addClass('exception');
                                                    for (var i = 0; i < exceptionObj[shipName][shipCode].length; i++) {
                                                        if (exceptionObj[shipName][shipCode][i] !== null || typeof exceptionObj[shipName][shipCode][i] !== 'undefined') {
                                                            exceptionstates.push(exceptionObj[shipName][shipCode][i]);
                                                        }
                                                    }
                                                    method.attr('data-exceptions', exceptionstates.join(','));
                                                } else {
                                                    method.addClass('no-exception');
                                                }
                                            } else {
                                                // Exception doesn't have separate shipping codes.
                                            }
                                        } else {
                                            // No exceptions found
                                        }
                                    });
                                }
                                catch (e) {
                                    printError(e);
                                }

                                $('.exception').addClass('hidden');
                                //$('.no-exception').find('.rdo-ship').prop('checked', true);
                                if (response.shipping_exceptions) {
                                    if (shipStateSelected) {
                                        $('.exception').each(function () {
                                            var self = $(this);
                                            var exceptionStates = self.data('exceptions').split(',');
                                            for (var i = 0; i < exceptionStates.length; i++) {
                                                if (shipStateSelected === exceptionStates[i]) {
                                                    self.removeClass('hidden');
                                                    $('.no-exception').addClass('hidden');
                                                    break;
                                                } else {
                                                    $('.no-exception').removeClass('hidden');
                                                }
                                            }
                                        });
                                    } else {
                                        $('.no-exception').removeClass('hidden');
                                    }
                                }

                                if ($('.rdo-ship:checked').parent('.shipping-method-item.hidden').length) {
                                    $('.shipping-method-item:not(.hidden):first').find('.rdo-ship').prop('checked', true);
                                }
                                cart.ajx.requestCartBalance($('.rdo-ship:checked').val());

                            } else {
                                printWarning('Response is NOT in json format.');
                            }
                            //cart.ajx.initEvents();
                            counter -= 1;
                        },
                        error: function () {
                          printError('Error fetching shipping information.');
                        }
                    });

                }, // end of initShippingMethods
                requestCartBalance: function (shipMethod) {
                    /*
                    Update shipping prices is run on initial page load,
                    and any updates to shipping methods, state, and/or zip code.
                    */
                    var counter = cart.config.counter;
                    counter += 1;
                    $.ajax({
                        url: cart.config.filepath,
                        beforeSend: function () {
                            $('#price_breakdown').addClass('attach-loader');
                        },
                        data: {
                            what_to_do: 'price_details',
                            shipping_method: shipMethod,
                            ship_state: cartElements.shipState.val(),
                            zip_code: cartElements.shipZipCode.val()
                        },
                        success: function (response) {
                            cart.utils.updateCartBalance(response);
                            cart.utils.updateShippingPrices(response);
                            cart.utils.updateCartPayments(response);
                            counter -= 1;
                        },
                        complete: function () {
                            $('#price_breakdown').removeClass('attach-loader');
                        },
                        error: function () {
                            printError('Error updating cart information.');
                        }
                    });
                }, // end of updatePrices

                /* === PROMO CODE === */
                applyPromo: function (code) {
                    if (cart.utils.getGrandTotal() > 0) {
                        var counter = cart.config.counter;
                        counter += 1;
                        $.ajax({
                            url: cart.config.filepath,
                            data: {
                                what_to_do: 'edit_promo_info',
                                shipping_method: cartElements.shipMethodsList.find('.rdo-ship:checked').val(),
                                ship_state: cartElements.shipState.val(),
                                zip_code: cartElements.shipZipCode.val(),
                                promo_code: code
                            },
                            success: function (response) {
                                var promoNameHTML = '',
                                    promoRemoveHTML = '',
                                    promoMsg = '';
                                if (response.promo.no_match) {
                                    // promo code is invalid
                                    //printError('Invalid promo code.');
                                    toastr.error('Invalid or expired promo code.');
                                    $('#promo_error').removeClass('hidden');
                                    $('.js-promo').addClass('hidden');
                                    $('#promo_msg').remove();
                                    cart.utils.promoErrorTimeout = setTimeout(function () {
                                        $('#promo_error').addClass('hidden');
                                    }, 2500);
                                } else {
                                    $('#promo_error').addClass('hidden');
                                    // Add promo to cart total (with delete button)
                                    $('.js-promo').removeClass('hidden');
                                    promoNameHTML += 'Discount(' + response.promo.promo_name + ')';
                                    promoRemoveHTML += ' <button class="ajx-del-promo btn btn--sm btn--gamma" type="button">&times;</button>';
                                    $('.ajx-promo-name').attr('data-promo-code', response.promo.promo_code).html(promoNameHTML + promoRemoveHTML);
                                    // Show promo message if promo applied successfully.
                                    promoMsg += 'Promotional code <strong>' + response.promo.promo_code + '</strong> was applied';
                                    promoMsg += "<br>";
                                    promoMsg += '<strong>' + response.promo.promo_name + '</strong> &dash; ' + response.promo.promo_description;

                                    if (!$('#promo_msg').length) {
                                        $('<div></div>').attr('id', 'promo_msg').addClass('msg success-msg').html(promoMsg).appendTo('#promo');
                                    } else {
                                        $('#promo_msg').html(promoMsg);
                                    }

                                    toastr.success('Promo code applied.');
                                }
                                cart.utils.updateCartBalance(response);
                                counter -= 1;
                            },
                            error: function () {
                                cart.errorHandler.throwError(cart.config.errorMessage);
                                printError('Promo code server error or timeout.');
                            }
                        });
                    } else {
                        toastr.warning('Your balance is already covered.', 'Unable to apply promo');
                    }
                }, // end of applyPromo
                deletePromo: function () {
                    var counter = cart.config.counter;
                    counter += 1;
                    $.ajax({
                        url: cart.config.filepath,
                        data: {
                            what_to_do: 'delete_promo_info',
                            shipping_method: $('.rdo-ship:checked').val(),
                            ship_state: cartElements.shipState.val(),
                            zip_code: cartElements.shipZipCode.val(),
                            promo_code: $('.ajx-promo-name').attr('data-promo-code')
                        },
                        success: function (response) {
                            // Hide promo row
                            $('.js-promo').toggleClass('hidden');
                            // Delete all promo info including promo messages.
                            $('#promo_msg').remove();
                            $('.ajx-promocode').val('');
                            $('.ajx-promo-name').attr({
                                'data-promo-code': ''
                            }).addClass('pull-left').html('No promotions were applied to this order.');
                            $('.ajx-promo-discount').text('0.00');
                            cart.utils.updateCartPayments(response);
                            cart.utils.updateCartBalance(response);
                            counter -= 1;

                            toastr.success('Promo code was removed.');
                        },
                        error: function () {
                            cart.errorHandler.throwError(cart.config.errorMessage);
                            printError('Promo code server error or timeout.')
                        }
                    });
                }, // end of deletePromo

                /* === PAYMENTS === */
                applyPayment: function (paymentType, passwordProtected) {
                    var counter = cart.config.counter,
                        requestType = 'GET',
                        dataToSend = {},
                        callbackFunc,
                        status, passwordForm;
                    counter += 1;

                    if (cart.utils.getGrandTotal() > 0) {
                        if (paymentType === 'gc') {
                            dataToSend = {
                                what_to_do: 'validate_gift_certificate_payment',
                                shipping_method: $('.rdo-ship:checked').val(),
                                ship_state: cartElements.shipState.val(),
                                zip_code: cartElements.shipZipCode.val(),
                                gc_key: $('.ajx-gc').val()
                            };
                            if (passwordProtected) {
                                dataToSend.gc_password = $('#gc_password').val();
                            }
                            callbackFunc = function (response) {
                                var gc = response.gc_po,
                                    paymentApplied = gc.flags.gc_po_payment_applied,
                                    requiresPassword = gc.flags.gc_po_passw_required,
                                    message = gc.mesage;
                                if (gc.status === true) {
                                    // GC payment applied successfully
                                    cart.utils.updateCartPayments(response);
                                    cart.utils.updateCartBalance(response);
                                    cart.utils.GCPOPassword.hideFor('gc');
                                    cart.utils.updateGCMessage(null, 'Gift certificate payment has been applied.');
                                } else {
                                    // GC payment didn't apply or applied with warnings
                                    if (paymentApplied === true) {
                                        // GC payment applied partially (show error message and apply as payment)
                                        cart.utils.updateGCMessage({ class: 'warning' }, message);
                                        cart.utils.updateCartPayments(response);
                                        cart.utils.updateCartBalance(response);
                                    } else {
                                        // PO payment did NOT apply. Show error message.
                                        cart.utils.updateGCMessage({ class: 'error' }, message);
                                    }
                                    // show hide password form
                                    if (requiresPassword === true) {
                                        cart.utils.GCPOPassword.showFor('gc');
                                    } else {
                                        cart.utils.GCPOPassword.hideFor('gc');
                                    }
                                }
                            };
                            requestType = 'GET';
                        } else if (paymentType === 'pd') {
                            dataToSend = {
                                what_to_do: 'validate_pd_payment',
                                plan_id: $('#plan_id').val(),
                                emplid: $('#employee_id').val(),
                                billing_type: $('#billing_type').val(),
                                shipping_method: $('.rdo-ship:checked').val(),
                                ship_state: cartElements.shipState.val(),
                                zip_code: cartElements.shipZipCode.val()
                            };
                            callbackFunc = function (response) {
                                // Display updated payroll balance
                                cart.utils.updatePayrollBalance(response);
                                cart.utils.updateCartPayments(response);
                            };
                            requestType = 'GET';
                        } else if (paymentType === 'po') {
                            dataToSend = {
                                what_to_do: 'validate_purchase_order_payment',
                                shipping_method: $('.rdo-ship:checked').val(),
                                ship_state: cartElements.shipState.val(),
                                zip_code: cartElements.shipZipCode.val(),
                                po_key: cartElements.poKey.val() // purchase order key value.
                            };
                            if (passwordProtected) {
                                dataToSend.po_password = $('#po_password').val();
                            }
                            callbackFunc = function (response) {
                                var po = response.gc_po,
                                    paymentApplied = po.flags.gc_po_payment_applied,
                                    requiresPassword = po.flags.gc_po_passw_required,
                                    message = po.mesage;
                                if (po.status === true) {
                                    // PO payment applied successfully
                                    cart.utils.updateCartPayments(response);
                                    cart.utils.updateCartBalance(response);
                                    cart.utils.GCPOPassword.hideFor('po');
                                    cart.utils.updatePOMessage(null, 'Purchase order payment has been applied.');
                                } else {
                                    // PO payment didn't apply or applied with warnings
                                    if (paymentApplied === true) {
                                        // PO payment applied partially (show error message and apply as payment)
                                        cart.utils.updatePOMessage({ class: 'warning' }, message);
                                        cart.utils.updateCartPayments(response);
                                        cart.utils.updateCartBalance(response);
                                    } else {
                                        // PO payment did NOT apply. Show error message.
                                        cart.utils.updatePOMessage({ class: 'error' }, message);
                                    }
                                    // show hide password form
                                    if (requiresPassword === true) {
                                        cart.utils.GCPOPassword.showFor('po');
                                    } else {
                                        cart.utils.GCPOPassword.hideFor('po');
                                    }
                                }
                            };
                            requestType = 'GET';
                        } else {
                            printError('Unknown payment type');
                            return;
                        }
                        $.ajax({
                            url: cart.config.filepath,
                            type: requestType,
                            data: dataToSend,
                            success: function (response) {
                                callbackFunc(response);
                                counter -= 1;
                            },
                            error: function (jqXHR, textStatus, errorThrown) {
                                cart.errorHandler.throwError(cart.config.errorMessage);
                                printError('Error applying ' + paymentType);
                                printError(textStatus);
                                printError(errorThrown);
                            }
                        });
                    } else {
                        toastr.warning('Please remove existing payment before applying new one.', 'Balance is already covered.');
                    }
                }, // end of applyPayment
                deletePayment: function () {
                    var counter = cart.config.counter;
                    var paymentId = $(this).parents('.payment-item').attr('data-payment-id');
                    var methodId = $(this).parents('.payment-item').attr('data-method-id');
                    counter += 1;
                    $(this).parents('.payment-item').remove();
                    $.ajax({
                        url: cart.config.filepath,
                        type: 'GET',
                        data: {
                            cancel_payment: paymentId,
                            //  Send methodID on payment cancellation
                            method_id: methodId,
                            shipping_method: $('.rdo-ship:checked').val(),
                            ship_state: cartElements.shipState.val(),
                            zip_code: cartElements.shipZipCode.val()
                        },
                        success: function (response) {
                            // toggle gift card message ONLY IF gc payment is deleted.
                            methodId = parseInt(methodId);
                            if (methodId === 2) {
                                //$('#gc_error').remove();
                                //$('#gc_success').remove();
                                cart.utils.GCPOPassword.hideFor('gc');
                                toastr.success('Gift certificate was removed as payment.');
                                //cart.utils.updateGCMessage(response);
                            }
                            // display updated payroll balance (if deleting PD payment) once removed
                            if (methodId === 3) {
                                cart.utils.updatePayrollBalance(response);
                                toastr.success('Payroll deduction was removed as payment.');
                            }
                            // PO - remove user messages
                            if (methodId === 5) {
                                // $('#po_error').remove();
                                // $('#po_success').remove();
                                cart.utils.GCPOPassword.hideFor('po');
                                toastr.success('Purchase order was removed as payment.');
                            }
                            counter -= 1;
                            cart.utils.updateCartPayments(response);
                            cart.utils.updateCartBalance(response);
                        },
                        error: function () {
                            cart.errorHandler.throwError(cart.config.errorMessage);
                            printError('Couldnt delete payment.');
                        }
                    });
                }, // end of deletePayment
                validatePaypalPayment: function (d) {
                    function showLoadingScreen () {
                        var loadScreen = $('<div></div>'),
                            loadMessage = $('<div></div>');
                        loadScreen.addClass('pp-loader');
                        loadMessage.addClass('pp-loader-message');
                        loadMessage.text('Loading Paypal...');
                        loadScreen.append(loadMessage);
                        $('body').append(loadScreen);
                    }
                    function removeLoadingScreen () {
                        if ($('.pp-loader').length) {
                            $('.pp-loader').remove();
                        }
                    }
                    var counter = cart.config.counter,
                        paypalData = {};
                    cart.utils.creditCardValidate(false);
                    // 2. Check if form fields are validated (frontend)
                    try {
                        if (cartElements.checkoutForm.valid()) {
                            counter += 1;
                            // 2.1 Prepare customer data to be sent to the server
                            paypalData = {
                                what_to_do: 'validate_paypal_payment',
                                first_name: $('#first_name').val(),
                                last_name: $('#last_name').val(),
                                billing_name: $('#billing_name').val(),
                                billing_address_1: $('#billing_address_1').val(),
                                billing_address_2: $('#billing_address_2').val(),
                                billing_city: $('#billing_city').val(),
                                billing_state: $('#billing_state').val(),
                                billing_zip_code: $('#billing_zip').val(),
                                phone: $('#phone').val(),
                                ship_name: $('#ship_name').val(),
                                address_1: $('#ship_address_1').val(),
                                address_2: $('#ship_address_2').val(),
                                city: $('#ship_city').val(),
                                state: $('#ship_state').val(),
                                zip_code: $('#ship_zip').val(),
                                ship_method: $('.rdo-ship:checked').val(),
                                email: $('#email').val()
                            };

                            // Overwrite paypal shipping data with instore shipping data if instore is a selected shipping method.
                            if (d.storePickup) {
                              paypalData.ship_name = $('#instore_ship_name').val();
                              paypalData.address_1 = $('#instore_ship_address_1').val();
                              paypalData.address_2 = $('#instore_ship_address_2').val();
                              paypalData.city = $('#instore_ship_city').val();
                              paypalData.state = $('#instore_ship_state').val();
                              paypalData.zip_code = $('#instore_zip_code').val();
                            }

                            // 2.2 Send prepared customer data to server
                            $.ajax({
                                url: cart.config.filepath,
                                type: 'GET',
                                data: paypalData,
                                beforeSend: showLoadingScreen,
                                success: function (response) {
                                    // If response format is other than JSON (could be a server error)
                                    if (typeof response === 'object') {
                                        // If server sends validation error
                                        if (response.error) {
                                            // Feedback to user.
                                            cart.errorHandler.throwError('[Validation error] ' + response.error);
                                            removeLoadingScreen();
                                        } else {
                                            // Submit data to paypal.
                                           cart.utils.populatePaypalInfo(response);
                                        }
                                    } else {
                                        cart.errorHandler.throwError(cart.config.errorMessage);
                                        removeLoadingScreen();
                                    }
                                    counter -= 1;
                                },
                                error: function () {
                                    cart.errorHandler.throwError(cart.config.errorMessage);
                                    removeLoadingScreen();
                                }
                            });
                        } else {
                            removeLoadingScreen();
                        }
                        cart.utils.creditCardValidate(true);
                    } catch (err) {
                        printError(err);
                    }
                },
                initEvents: function () {
                    var selectedShipMethod = $('.rdo-ship:checked').val();
                    cart.ajx.requestCartBalance(selectedShipMethod);
                }, // end of initEvents

                requestInstoreInfo: function(instore_zipcode){
                    console.log("Call ajax.request.");
/*
                    var counter = cart.config.counter;
                    counter += 1;
                    $.ajax({
                        url: cart.config.filepath,
                        data: {
                            what_to_do: "request-instore_info",
                            instore_zip_code: instore_zip_code
                        },

                        success: function (response) {
                            counter -= 1;
                        },

                        error: function () {
                            printError('Error requestInstore function.');
                        }
                    });
*/
                }
            }, // end of ajx

            /* === CART INITIALIZATION === */
            init: function () {

                function initNotificationOptions() {
                    toastr.options.closeButton = true;
                    toastr.options.showDuration = 200;
                    toastr.options.closeDuration = 100;
                    toastr.options.preventDuplicates = true;
                    toastr.options.positionClass = 'toast-top-right';
                    // toastr.options.showMethod = 'slideDown';
                    // toastr.options.hideMethod = 'slideUp';
                    // toastr.options.closeMethod = 'slideUp';
                }

                initNotificationOptions();

                var shippingMethodSelected = function () {
                    return $('.shipping-method-item:not(.hidden)').find('.rdo-ship:checked').length;
                };
                /* === Initial Load === */

                cart.utils.lock(cart.config.counter);

                //cart.ajx.initShippingMethods(cartElements.billState.val());
                cart.utils.copyToShipping();
                cart.utils.confirmPayrollElibility();
                cart.utils.setCreditCardExpiry();
                /* === Event triggers === */
                cartElements.inputs.on('blur', cart.utils.trimInput);
                cartElements.newCustomerChk.on('change', cart.utils.clearPassword);
                cartElements.copyToShippingChk.on('change', cart.utils.copyToShipping);
                cartElements.paymentSelector.on('click focus', cart.utils.togglePaymentMethods);
                /* === Ajax event triggers === */

                cartElements.shippingSelector.on('change', cart.ajx.initEvents);
                // When selecting shipping methods, check if shipping method is 'INSTORE'
                cartElements.shippingSelector.on('change.instore', function () {
                  var shipMethod = $(this),
                      shipMethodType = shipMethod.val();
                  console.log(shipMethodType);
                  if (shipMethodType.toString() === 'INSTORE') {
                    $('#store_list').removeClass('hidden');
                    $('#store_list').find('input[type="hidden"]').attr('disabled', false);
                    $('#customer_shipping_info').addClass('hidden');
                    $('#shipment_form, sp_shipment_form').find('.form-ctrl').attr('disabled', true);
                  } else {
                    $('#store_list').addClass('hidden');
                    $('#store_list').find('input[type="hidden"]').attr('disabled', true);
                    $('#customer_shipping_info').removeClass('hidden');
                    $('#shipment_form, sp_shipment_form').find('.form-ctrl').attr('disabled', false);
                  }
                });

                cartElements.shippingSelector.on('change.validate', function () {
                    if (shippingMethodSelected()) {
                        cart.errorHandler.hideShippingError();
                    } else {
                        cart.errorHandler.throwShippingError();
                    }
                });
                /*
                    === Promo code ===
                */

                // toggle promo
                if (cartElements.promoToggle.length) {
                    cartElements.promoToggle.on('change', function() {
                        var promoDiv = cartElements.promoDiv,
                            promoError = $('#promo_error'),
                            promoField = cartElements.promoField;
                        if (promoDiv.hasClass('hidden')) {
                            promoDiv.removeClass('hidden');
                        } else {
                            promoDiv.addClass('hidden');
                            // hide error when unchecked
                            if (cart.utils.promoErrorTimeout !== null) {
                                clearTimeout(cart.utils.promoErrorTimeout);
                            }
                            promoError.addClass('hidden');
                            // clear promo field when unchecked
                            promoField.val('');
                        }
                    });
                }
                //cartElements.applyPromoBtn.on('click', cart.ajx.applyPromo);
                cartElements.promoField.on('blur', function () {
                    var code = $(this).val();
                    if (code !== '') {
                        cart.ajx.applyPromo(code);
                    }
                });
                cartElements.cartSummary.on('click', '.ajx-del-promo', cart.ajx.deletePromo);

                /*
                    === Gift certificates ===
                */

                // apply gift certificates
                cartElements.gcField.on('blur', function () {
                    var code = $(this).val();
                    if (code !== '') {
                        cart.ajx.applyPayment('gc', false);
                    }
                });
                // validate gift certificate password
                $('#validate_gc_password').on('click', function () {
                    // set flag to 'true' if password protected
                    cart.ajx.applyPayment('gc', true);
                });

                // cartElements.applyGCButton.on('click', function () {
                //     cart.ajx.applyPayment('gc');
                // });

                /*
                    === Payroll Deduction ===
                */
                cartElements.pdEligibleChk.on('change', cart.utils.confirmPayrollElibility);
                cartElements.pdApplyBtn.on('click', function () {
                    cart.ajx.applyPayment('pd');
                });

                /*
                    === PayPal ===
                */
                cartElements.paypalBtn.on('click', function () {
                    if (shippingMethodSelected()) {
                        cart.errorHandler.hideShippingError();
                        if ($('.rdo-ship:checked').val() === 'INSTORE') {
                            cart.ajx.validatePaypalPayment({
                              storePickup: true
                            });
                        } else {
                            cart.ajx.validatePaypalPayment({
                              storePickup: false
                            });
                        }
                    } else {
                        cart.errorHandler.throwShippingError();
                    }
                });

                /*
                    === Purchase orders ===
                */
                // apply purchase orders
                cartElements.poApplyBtn.on('click', function () {
                    // set flag to 'false' if not password protected
                    cart.ajx.applyPayment('po', false);
                });

                // validate purchase order password
                $('#validate_po_password').on('click', function () {
                    // set flag to 'true' if password protected
                    cart.ajx.applyPayment('po', true);
                });
                // delete payment
                cartElements.cartSummary.on('click', '.ajx-del-payment', cart.ajx.deletePayment);

                /*
                    === In-store pickup ===
                */
                cartElements.instoreList.find('li').each(function () {
                  var instoreListItem = $(this);

                  function prepopulateShippingInfo () {
                    if (instoreListItem.hasClass('selected')) {
                      $('#instore_ship_name').val(instoreListItem.data('shipName'));
                      $('#instore_ship_address_1').val(instoreListItem.data('shipAddress'));
                      $('#instore_ship_city').val(instoreListItem.data('shipCity'));
                      $('#instore_ship_state').val(instoreListItem.data('shipState'));
                      $('#instore_zip_code').val(instoreListItem.data('shipZip'));
                    }
                  }

                  prepopulateShippingInfo();

                  instoreListItem.on('click', function () {
                      console.log('Called instore radio button');

                    $(this)
                      .siblings().removeClass('selected')
                      .end()
                      .addClass('selected');

                      prepopulateShippingInfo();
                      console.log( "instore_zip code: " + $('#instore_zip_code').val() );
                      var selected_instore_zip = $('#instore_zip_code').val();
                      cart.ajx.requestInstoreInfo(selected_instore_zip);

                  });
                });
            }
        }; // end of cart object

        cart.init();

    }; // end of initSecureCheckout

    // For faster page load, functions are not invoked unless needed.
    if (isShoppingCart) {
        // 1. Shopping Cart Ajax
            // Source: ../modules/ajax.js
        ajaxLib.shoppingCart.init();
    } else if (isSecureCheckout) {
        // 2. Secure Checkout Ajax
            // Source: internal **
                // ** checkout is important and complex
                // and shouldn't be moved to external file yet (not to break it)
        initSecureCheckout();
    }

}());
```
