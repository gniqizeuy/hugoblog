---
weight: 1
bookFlatSection: true
title: "Example Site"
---

# Introduction

## A blog powered by hugo

Lorem markdownum, a quoque nutu est *quodcumque mandasset* veluti. Passim
inportuna totidemque nympha fert; repetens pendent, poenarum guttura sed vacet
non, mortali undas. Omnis pharetramque gramen portentificisque membris servatum
novabis fallit de nubibus atque silvas mihi. **Dixit repetitaque Quid**; verrit
longa; sententia [mandat](http://pastor-ad.io/questussilvas) quascumque nescio
solebat [litore](http://lacrimas-ab.net/); noctes. *Hostem haerentem* circuit
[plenaque tamen](http://www.sine.io/in).

## Features

- Clean simple design
- Light and Mobile-Friendly
- Multi-language support
- Customisable
- Zero initial configuration
- Handy shortcodes
- Comments support
- Simple blog and taxonomy
- Primary features work without JavaScript
- Dark Mode

## Fortran

Fortran is a general-purpose, compiled imperative programming language that is especially suited to numeric computation and scientific computing.


    ! Cooley-Tukey FFT
    recursive subroutine fft(x,sgn)
    
        implicit none
        integer, intent(in) :: sgn
        complex(8), dimension(:), intent(inout) :: x
        complex(8) :: t
        integer :: N
        integer :: i
        complex(8), dimension(:), allocatable :: even, odd
    
        N=size(x)
    
        if(N .le. 1) return
    
        allocate(odd((N+1)/2))
        allocate(even(N/2))
    
        ! divide
        odd =x(1:N:2)
        even=x(2:N:2)
    
        ! conquer
        call fft(odd, sgn)
        call fft(even, sgn)
    
        ! combine
        do i=1,N/2
            t=exp(cmplx(0.0d0,sgn*2.0d0*pi*(i-1)/N))*even(i)
            x(i)     = odd(i) + t
            x(i+N/2) = odd(i) - t
        end do
    
        deallocate(odd)
        deallocate(even)

    end subroutine fft

